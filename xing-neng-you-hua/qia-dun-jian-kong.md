[链接](http://mrpeak.cn/blog/ui-detect/)

# 1.runloop 检测



我们可以假设这样一套机制：每隔16ms让UI线程来报道一次，如果16ms之后UI线程没来报道，那就一定是在执行某个耗时的任务。这种抽象的描述翻译成代码，可以用如下表述：

**我们启动一个worker线程，worker线程每隔一小段时间（delta）ping一下主线程（发送一个NSNotification），如果主线程此时有空，必然能接收到这个通知，并pong以下（发送另一个NSNotification），如果worker线程超过delta时间没有收到pong的回复，那么可以推测UI线程必然在处理其他任务了，此时我们执行第二步操作，暂停UI线程，并打印出当前UI线程的函数调用栈**

难点在这第二步，如何暂停UI线程，同时获取到callstack。

iOS的多线程编程一般使用NSOperation或者GCD，这两者都无法暂停每个正在执行的线程。所谓的cancel调用也只能在目标线程空闲的时候，主动检测cancelled状态，然后主动sleep，这显然非我所欲。

还剩下pthread一途，pthread系列api当中有个函数pthread_kill()看起来符合期望。





# 2.`CADisplayLink`进行检测	

详见[YYFPSLabel](https://github.com/yehot/YYFPSLabel)

 实现思路：

- `CADisplayLink` 默认每秒 60次；
- 将 `CADisplayLink` add 到 `mainRunLoop` 中；
- 使用 `CADisplayLink` 的 `timestamp` 属性，在 `CADisplayLink` 每次 tick 时，记录上一次的 `timestamp`；
- 用 _count 记录 `CADisplayLink` tick 的执行次数;
- 计算此次 tick 时， `CADisplayLink` 的当前 timestamp 和 _lastTimeStamp 的差值；
- 如果差值大于1，fps = _count / delta，计算得出 FPS 数；



知识点:使用NSProxy防止CADisplayLink 加入 target 的时候循环引用， `__weak typeof(self) weakSelf = self`也不能解决 displayLinkWithTarget 循环引用的问题, 猜测 NSTimer 的 tagrget 中，对 weakSelf 又进行了类似 StrongSelf 操作

```swift
@interface YYWeakProxy : NSProxy
```