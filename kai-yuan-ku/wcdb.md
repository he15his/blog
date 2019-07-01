1

# 线程安全和性能

## 线程安全

在讲线程安全和性能前，必须要了解 `SQLite` 是怎么实现线程安全和达到高性能，具体可以参考[《SQLite 线程安全和并发》](https://github.com/xiangwangfeng/xiangwangfeng.github.io/wiki/SQLite-线程安全和并发)。使用 `SQLite` 时常规的优化方案无非是

- 缓存 `sqlite3_prepare` 编译结果
- 使用 `WAL` 模式
- 采用多线程模式，单写多读
- 合理安排事务



简单来说，`WCDB` 的连接池通过读写锁保证线程安全，和 `FMDatabasePool` 使用 `gcd queue` 并没有太多差异，一些肉眼可见的区别在于

- `WCDB` 并不对外暴露数据库连接对象，以减少外面错误使用的几率。
- `WCDB` 在连接池之外还提供基于 `ThreadLocal` 的缓存机制，保证当前事务操作下永远只使用同一个连接。 (详见 `Database::flowOut`)
- 内部自动约束并发数，并对不合理的并发做出提示。比如连接数超过 `std::thread::hardware_concurrency()` 就会有警告。 (详见 `HandlePool::flowOut`)
- 连接回收基于 `C++` 变量作用域。这一点上在我看倒没有明显的优劣点，反倒有点炫技的成分，为了实现这一点还需要额外引入 `RecylceHandle`。
- 支持内存不足时的数据库连接自动回收。 (详见 `Database::purgeFreeHandles`)

这些细微的差别能够使得 `WCDB` 在保证线程安全和合理并发的前提下，使用起来更加方便安心。



## 性能

除了上面说的合理设计框架，合理提供并发外，`WCDB` 还做了一些额外性能有点。下面仅列出一些我读代码和 `wiki` 的发现。

- `checkpointing` 优化

在使用 `WAL` 模式时，默认情况下，当 `WAL 文件` 大小超过 `1000` 个页大小时，`SQLite` 就会尝试将 `WAL 文件` 写回数据库文件，这就是所谓的 `checkpointing`。(详见 [wal](https://www.sqlite.org/wal.html)) 那么在大量数据批量写入的场景下，可能会不停的产生提交文件到数据库的事务。而 `WCDB` 的做法则是在触发 checkpointing 时，通过延时队列进行，避免大量写入时不停的触发 `WalCheckpoint` 调用。

代码如下

```
[](std::shared_ptr<Handle> &handle, Error &error) -> bool {
             handle->registerCommittedHook(
                 [](Handle *handle, int pages, void *) {
                     static TimedQueue<std::string> s_timedQueue(2);
                     if (pages > 1000) {
                         s_timedQueue.reQueue(handle->path);
                     }
                     static std::thread s_checkpointThread([]() {
                         pthread_setname_np(
                             ("WCDB-" + Database::defaultCheckpointConfigName)
                                 .c_str());
                         while (true) {
                             s_timedQueue.waitUntilExpired(
                                 [](const std::string &path) {
                                     Database database(path);
                                     WCDB::Error innerError;
                                     database.exec(StatementPragma().pragma(
                                                       Pragma::WalCheckpoint),
                                                   innerError);
                                 });
                         }
                     });
                     static std::once_flag s_flag;
                     std::call_once(s_flag,
                                    []() { s_checkpointThread.detach(); });
                 },
                 nullptr);
             return true;
         },
```

通过 `TimedQueue` 将同个数据库的 `WalCheckpoint` 合并延迟到 2 秒后统一进行。

- `SQLITE_BUSY` 优化

`SQLite` 的机制并不允许进行多线程同时进行写操作，当发生多个线程进行写操作时未得到锁的那一方将直接返回 `SQLITE_BUSY`。从 `FMDB` 的提交记录我们可以看出，`ccgus` 对怎么处理 `SQLITE_BUSY` 也是相当头疼，具体可以参考 `FMDB` 中关于 `SQLITE_BUSY` 的 `issues`。目前 `FMDB` 的做法是默认重试 `2` 秒，在此期间调用 `sqlite3_sleep` 随机休眠几十毫秒，等待另外一个线程释放锁。这种处理方式可以较大程度上缓解 `SQLITE_BUSY` 的问题，但仍不可避免。这也是 `WCDB Benchmark` 认为 `FMDB` 无法支持 `Multi-Thread WriteWrite` 的原因。

而 `WCDB` 的处理方式则相当粗暴：通过修改 `sqlcipher` 源码，如果当前未进入事务状态而产生 `SQLITE_BUSY` 则会挂起等待，超时时间为 `10` 秒。详细代码可以参见 `btree.c` 中的 `sqlite3BtreeBeginTrans` 方法。

```C++
do {
    //一堆判断
    sqlite3PagerBegin(pBt->pPager,wrflag>1,sqlite3TempInMemory(p->db));
    //一堆判断
}while( (rc&0xFF)==SQLITE_BUSY && pBt->inTransaction==TRANS_NONE &&
          btreeInvokeBusyHandler(pBt) );
```

- 编译选项优化

`SQLite` 有大量预编译宏选项可以配置，具体可以参见 `sqliteLimit.h` 和 `sqliteInt.h`，`WCDB` 也对此作了较多配置，具体可以参考 `sqlchiper-preprocessed.xcodeproj` 中的宏定义。像我在 [《SQLite 分表》](https://github.com/xiangwangfeng/xiangwangfeng.github.io/wiki/SQLite-分表) 提到的 `SQLITE_MALLOC_SOFT_LIMIT` 就是偷师自微信，通过设置它为 0，可以加快在大量表情况下的初始化过程。从微信分享给出的资料还有相当多的优化项，如 `开启 mmap`，禁用文件锁（针对 `iOS`单进程的场景）等，具体可以参考 [《微信iOS SQLite源码优化实践》](https://dev.qq.com/topic/57b6a449433221be01499486) 并查找对应源码进行对照。