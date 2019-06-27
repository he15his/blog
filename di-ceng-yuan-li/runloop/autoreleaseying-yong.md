### AutoreleasePool

App启动后，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 \_wrapRunLoopWithAutoreleasePoolHandler\(\)。

第一个 Observer 监视的事件是 Entry\(即将进入Loop\)，其回调内会调用 \_objc\_autoreleasePoolPush\(\) 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

第二个 Observer 监视了两个事件： BeforeWaiting\(准备进入休眠\) 时调用\_objc\_autoreleasePoolPop\(\) 和 \_objc\_autoreleasePoolPush\(\) 释放旧的池并创建新池；Exit\(即将退出Loop\) 时调用 \_objc\_autoreleasePoolPop\(\) 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。



### 事件响应

苹果注册了一个 Source1 \(基于 mach port 的\) 用来接收系统事件，其回调函数为 \_\_IOHIDEventSystemClientQueueCallback\(\)。

当一个硬件事件\(触摸/锁屏/摇晃等\)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考[这里](http://iphonedevwiki.net/index.php/IOHIDFamily)。SpringBoard 只接收按键\(锁屏/静音等\)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 \_UIApplicationHandleEventQueue\(\) 进行应用内部的分发。

\_UIApplicationHandleEventQueue\(\) 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。

