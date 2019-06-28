

经测试iOS最多能创建66个线程, 66 = 64 \( 最大名称线线程池大小\) + 主线程+ 一些其他随机non-GCD线程



```
for (int i=1; i<30000; i++) {
 dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
 [NSThread sleepForTimeInterval:100000];
 });
}
```

多线程的并发是用时间片轮转等方法实现的，iPhone 的 CPU 只有两个核心，所以过多线程同时存在仍然可能会造成对主线程的影响。另外线程创建、销毁、上下文切换等也会消耗很多 CPU 资源。有些任务内部会有全局的锁（比如 readme 里提到的 CoreText 绘制时的 CGFont 内部锁），这样增加并发并没有作用，反而会带来诸如 readme 中那样的问题。

