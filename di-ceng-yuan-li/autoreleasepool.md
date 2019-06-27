# AutoreleasePool

App 中有多少个 AutoreleasePool ?

AutoreleasePool 可以嵌套吗？

一个对象可以放到一个 pool 中多次吗？

AutoreleasePool 与 多线程 有什么关系？

### 有什么用？ {#有什么用？}

当调用 \[obj autorelease\] 时，不会立刻释放 obj ，而是会把 obj 保存到 AutoreleasePool 中，当 AutoreleasePool pop 时，统一调用 \[obj release\]

方便对象的内存管理。

可以延长 obj 的生命周期。

#### 自己的代码 {#自己的代码}

自己在代码中写 AutoreleasePool 可以减小内存峰值。

程序中循环遍历时有大量临时变量的时候最好手动创建。

参考：[https://developer.apple.com/documentation/foundation/nsautoreleasepool?language=objc](https://developer.apple.com/documentation/foundation/nsautoreleasepool?language=objc)

### 如何使用 {#如何使用}

#### 主线程的每个Runloop周期会创建一个新的 AutoreleasePool {#主线程的每个runloop周期会创建一个新的-autoreleasepool}

```
App启动后，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。
第一个 Observer 监视的事件是 Entry(即将进入Loop)，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

第二个 Observer 监视了两个事件： 
    BeforeWaiting(准备进入休眠) 时调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；
    Exit(即将退出Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。
在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。

main runloop
/*
    observers = (
    // kCFRunLoopEntry
    "<CFRunLoopObserver 0x2815d8500 [0x241acd610]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = <redacted> (0x23eb9dec0), context = <CFArray 0x282a94e70 [0x241acd610]>{type = mutable-small, count = 1, values = (\n\t0 : <0x1063b8058>\n)}}",

    // kCFRunLoopBeforeWaiting
    "<CFRunLoopObserver 0x2815dc5a0 [0x241acd610]>{valid = Yes, activities = 0x20, repeats = Yes, order = 0, callout = <redacted> (0x23e7b2f28), context = <CFRunLoopObserver context 0x280fddea0>}",

    // kCFRunLoopBeforeWaiting || kCFRunLoopExit
    "<CFRunLoopObserver 0x2815d8640 [0x241acd610]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 1999000, callout = <redacted> (0x23ebcd77c), context = <CFRunLoopObserver context 0x106403e80>}",
    "<CFRunLoopObserver 0x2815d81e0 [0x241acd610]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = <redacted> (0x215f94cd4), context = <CFRunLoopObserver context 0x0>}",
    "<CFRunLoopObserver 0x2815d85a0 [0x241acd610]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2001000, callout = <redacted> (0x23ebcd7fc), context = <CFRunLoopObserver context 0x106403e80>}",
    "<CFRunLoopObserver 0x2815d8460 [0x241acd610]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = <redacted> (0x23eb9dec0), context = <CFArray 0x282a94e70 [0x241acd610]>{type = mutable-small, count = 1, values = (\n\t0 : <0x1063b8058>\n)}}"
    ),
*/

/*
    typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    1 kCFRunLoopEntry = (1UL << 0),
    2 kCFRunLoopBeforeTimers = (1UL << 1),
    4 kCFRunLoopBeforeSources = (1UL << 2),
    32 kCFRunLoopBeforeWaiting = (1UL << 5),
    64 kCFRunLoopAfterWaiting = (1UL << 6),
    128 kCFRunLoopExit = (1UL << 7),
    kCFRunLoopAllActivities = 0x0FFFFFFFU
*/
```

#### 使用容器的block版本的枚举器时，内部会自动添加一个AutoreleasePool： {#使用容器的block版本的枚举器时，内部会自动添加一个autoreleasepool：}

```
[array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    // 这里被一个局部@autoreleasepool包围着
}];
```

当然，在普通for循环和for in循环中没有，所以，还是新版的block版本枚举器更加方便。for循环中遍历产生大量autorelease变量时，就需要手加局部AutoreleasePool

#### 自己的代码中使用 {#自己的代码中使用}

```
@autoreleasepool {
    //
}
```

### 实现原理 {#实现原理}

#### 编译如何处理 {#编译如何处理}

代码：

```
@autoreleasepool {
    int a = 1;
    int b = a;
}
```

命令：

```
生成汇编
clang -S -fobjc-arc main.m -o main.s
生成目标文件
clang -fmodules -c main.m -o main.o
```

编译后的代码：

```
void* pool = objc_autoreleasePoolPush();
// do sth
objc_autoreleasePoolPop(pool);
```

更进一步：

```
void *objc_autoreleasePoolPush(void) {
    return AutoreleasePoolPage::push();
}
void objc_autoreleasePoolPop(void *pool) {
    AutoreleasePoolPage::pop(pool);
}
原理：
- 调用 AutoreleasePoolPage::push() 会返回 一个哨兵对象的地址，这个地址就是 一个 pool 的开端
- 当对象调用 autorelease 方法时，会将 引用计数+1 的对象加入 AutoreleasePoolPage 的栈中
- 调用 AutoreleasePoolPage::pop 方法会向栈中的对象发送 release 消息
```

### 运行时如何处理 {#运行时如何处理}

#### AutoreleasePoolPage {#autoreleasepoolpage}

```
class AutoreleasePoolPage {
    static pthread_key_t const key = AUTORELEASE_POOL_KEY;  // 唯一标识
    magic_t const magic;
    id *next;   // 指向最新添加的 autoreleased 对象的下一个位置，初始化时指向 begin() 
    pthread_t const thread; // 当前page所在的线程
    AutoreleasePoolPage * const parent; // 双向链表parent
    AutoreleasePoolPage *child; // 双向链表child
    uint32_t const depth;
    uint32_t hiwat;
    static size_t const SIZE = PAGE_MAX_SIZE;  // 4096 bytes
    static size_t const COUNT = SIZE / sizeof(id);  // 指针数量
    // method
    static inline AutoreleasePoolPage *hotPage();
    static inline AutoreleasePoolPage *coldPage();
    static inline void *push();
    static inline id *autoreleaseFast(id obj);
    id *autoreleaseFullPage(id obj, AutoreleasePoolPage *page);
    static inline void pop(void *token);
};
#define POOL_SENTINEL nil   // 哨兵
```

#### 从编译到运行时 {#从编译到运行时}

```
static id objc_retainAutoreleaseAndReturn(id obj) {
    return objc_retainAutorelease(obj);
}
id objc_retainAutorelease(id obj) {
    return objc_autorelease(objc_retain(obj));
}
id objc_autoreleaseReturnValue(id obj) {
    return objc_autorelease(obj);
}
id objc_autorelease(id obj) {
    if (!obj) return obj;
    if (obj-
>
isTaggedPointer()) return obj;
    return obj-
>
autorelease();
}
objc_object::autorelease() {
    // UseGC is allowed here, but requires hasCustomRR.
    assert(!UseGC  ||  ISA()-
>
hasCustomRR());
    if (isTaggedPointer()) return (id)this;
    if (! ISA()-
>
hasCustomRR()) return rootAutorelease();
    return ((id(*)(objc_object *, SEL))objc_msgSend)(this, SEL_autorelease);
}
// NSObject.mm
static inline id autorelease(id obj)
{
    assert(obj);
    assert(!obj-
>
isTaggedPointer());
    id *dest __unused = autoreleaseFast(obj);   // 调用 
    assert(!dest  ||  *dest == obj);
    return obj;
}
```

#### AutoreleasePoolPage::push\(\) {#autoreleasepoolpagepush}

```
static inline void *push() {
   return autoreleaseFast(POOL_SENTINEL);   // POOL_SENTINEL为哨兵
}
/*
    有 hotPage 并且 page 不满
        调用 page-
>
add(obj) 方法将对象添加至 AutoreleasePoolPage 的栈中
    有 hotPage 并且当前 page 已满，调用 autoreleaseFullPage 
        1. 初始化一个新的页(如果有child，直接复用)
        2.调用 page-
>
add(obj) 方法将对象添加至 AutoreleasePoolPage 的栈中
        3.设置新的hotPage
    无 hotPage，autoreleaseNoPage
        1.调用 autoreleaseNoPage 创建一个 page
        2.调用 page-
>
add(obj) 方法将对象添加至 AutoreleasePoolPage 的栈中
        3.设置hotPage
*/
static inline id *autoreleaseFast(id obj)
{
   AutoreleasePoolPage *page = hotPage();
   if (page 
&
&
 !page-
>
full()) {
       return page-
>
add(obj);
   } else if (page) {
       return autoreleaseFullPage(obj, page);
   } else {
       return autoreleaseNoPage(obj);
   }
}
// Check per-thread single page
static inline AutoreleasePoolPage *hotPage() {
    AutoreleasePoolPage *result = (AutoreleasePoolPage *)
        tls_get_direct(key);
    if (result) result-
>
fastcheck();
    return result;
}
// 返回 obj 的内存地址 
id *add(id obj) {
    assert(!full());
    unprotect();
    id *ret = next;  
    *next++ = obj;
    protect();
    return ret;
}
// hotPage已满
static id *autoreleaseFullPage(id obj, AutoreleasePoolPage *page) {
    do {
        if (page-
>
child) page = page-
>
child;
        else page = new AutoreleasePoolPage(page);
    } while (page-
>
full());
    setHotPage(page);
    return page-
>
add(obj);
}
// 没有hotPage
static id *autoreleaseNoPage(id obj) {
    AutoreleasePoolPage *page = new AutoreleasePoolPage(nil);
    setHotPage(page);
    if (obj != POOL_SENTINEL) {
        page-
>
add(POOL_SENTINEL);
    }
    return page-
>
add(obj);
}
static inline void setHotPage(AutoreleasePoolPage *page) {
    if (page) page-
>
fastcheck();
    tls_set_direct(key, (void *)page);
}
```

#### AutoreleasePoolPage::pop\(ctxt\); {#autoreleasepoolpagepopctxt}

遍历双向链表中的所有page对象，对page中存储的obj 出栈 分别做release操作

    /*
    1. page 中的 [obj release]
    2. 删除 page
    */
    static inline void pop(void *token) {
        AutoreleasePoolPage *page = pageForPointer(token);
        id *stop = (id *)token;
        // release
        page-
    >
    releaseUntil(stop);
        // delete child page
        if (page-
    >
    child) {
            if (page-
    >
    lessThanHalfFull()) {
                page-
    >
    child-
    >
    kill();
            } else if (page-
    >
    child-
    >
    child) {
                page-
    >
    child-
    >
    child-
    >
    kill();
            }
        }
    }
    void releaseUntil(id *stop) {
        // Not recursive: we don't want to blow out the stack 
        // if a thread accumulates a stupendous amount of garbage
        while (this-
    >
    next != stop) {
            // Restart from hotPage() every time, in case -release 
            // autoreleased more objects
            AutoreleasePoolPage *page = hotPage();
            // fixme I think this `while` can be `if`, but I can't prove it
            while (page-
    >
    empty()) { // 如果为空 则
                page = page-
    >
    parent;
                setHotPage(page);
            }
            page-
    >
    unprotect();
            id obj = *--page-
    >
    next;
            memset((void*)page-
    >
    next, SCRIBBLE, sizeof(*page-
    >
    next));
            page-
    >
    protect();
            if (obj != POOL_SENTINEL) {
                objc_release(obj);
            }
        }
        setHotPage(this);
    }
    void kill() {
        // Not recursive: we don't want to blow out the stack 
        // if a thread accumulates a stupendous amount of garbage
        AutoreleasePoolPage *page = this;
        while (page-
    >
    child) page = page-
    >
    child; // 找到最后一个 page
        AutoreleasePoolPage *deathptr;
        do {
            deathptr = page;
            page = page-
    >
    parent;
            if (page) {
                page-
    >
    unprotect();
                page-
    >
    child = nil;  
                page-
    >
    protect();
            }
            delete deathptr;
        } while (deathptr != this);
    }

#### NSOject {#nsoject}

```
static inline id autorelease(id obj)
{
    assert(obj);
    assert(!obj-
>
isTaggedPointer());
    id *dest __unused = autoreleaseFast(obj);   // 此处会把 obj push 到 page 中
    assert(!dest  ||  *dest == obj);
    return obj;
}
```

整个App的pool 底层都使用一个双向链表实现

自动释放池是由 AutoreleasePoolPage 以双向链表的方式实现的。

每一个自动释放池都是由一系列的 AutoreleasePoolPage 组成，并且每一个 AutoreleasePoolPage 的大小都是 4096 字节。

App中的所有自动释放池都存储在同一个双向链表中，一个自动释放池的开头是 哨兵对象的，存储了一个nil对象。

### 与多线程的关系？ {#与多线程的关系？}

主线程中的自动释放池是自动创建的。实际上，我们常用的多线程管理方式\(GCD, NSOprationQueue, NSThread\)也都会帮我们创建对应的AutoreleasePool \(当第一次 push obj 的时候创建\) 。在线程结束时，执行 pop 。

具体参考：[https://stackoverflow.com/questions/24952549/does-nsthread-create-autoreleasepool-automatically-now](https://stackoverflow.com/questions/24952549/does-nsthread-create-autoreleasepool-automatically-now)

注意：如果一个子线程一直在运行，而不会结束时， AutoreleasePool 不会及时的 pop。

```
/***********************************************************************
   Autorelease pool implementation
   A thread's autorelease pool is a stack of pointers. 
   Each pointer is either an object to release, or POOL_SENTINEL which is 
     an autorelease pool boundary.
   A pool token is a pointer to the POOL_SENTINEL for that pool. When 
     the pool is popped, every object hotter than the sentinel is released.
   The stack is divided into a doubly-linked list of pages. Pages are added 
     and deleted as necessary. 
   Thread-local storage points to the hot page, where newly autoreleased 
     objects are stored. 
**********************************************************************/
BREAKPOINT_FUNCTION(void objc_autoreleaseNoPool(id obj));
```

#### TLS-线程局部存储 {#tls-线程局部存储}

在 hotPage\(\) 和 setHotPage\(\) 的实现中有两个底层的函数：

```
static pthread_key_t const key = AUTORELEASE_POOL_KEY;  // 唯一标识
tls_set_direct(key, (void *)page);
AutoreleasePoolPage *result = (AutoreleasePoolPage *)tls_get_direct(key);
```

tls \(Thread Local Storage\) 线程局部存储，用来将数据与一个正在运行的线程关联起来。

进程中的 全局变量 和 函数内定义的 静态变量\(static\) 是各线程都可以访问的共享变量。在一个线程内修改，对所有线程生效。

TLS技术的作用：为了避免多个线程同时访存同一全局变量或者静态变量时所导致的冲突，尤其是多个线程同时需要修改这一变量时。为了解决这个问题，可以通过TLS机制，为每一个使用该全局变量的线程都提供一个变量值的副本，每一个线程均可以独立地改变自己的副本，而不会和其它线程的副本冲突。从线程的角度看，就好像每一个线程都完全拥有该变量。而从全局变量的角度上来看，就好像一个全局变量被克隆成了多份副本，而每一份副本都可以被一个线程独立地改变。

### 参考 {#参考}

* [https://draveness.me/autoreleasepool](https://draveness.me/autoreleasepool)
* [https://blog.sunnyxx.com/2014/10/15/behind-autorelease/](https://blog.sunnyxx.com/2014/10/15/behind-autorelease/)



