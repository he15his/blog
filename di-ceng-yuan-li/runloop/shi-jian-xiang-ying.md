### 事件响应

苹果注册了一个 Source1 \(基于 mach port 的\) 用来接收系统事件，其回调函数为 \_\_IOHIDEventSystemClientQueueCallback\(\)。

当一个硬件事件\(触摸/锁屏/摇晃等\)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考[这里](http://iphonedevwiki.net/index.php/IOHIDFamily)。SpringBoard 只接收按键\(锁屏/静音等\)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 \_UIApplicationHandleEventQueue\(\) 进行应用内部的分发。

\_UIApplicationHandleEventQueue\(\) 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。

