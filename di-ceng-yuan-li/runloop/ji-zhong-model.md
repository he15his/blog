kCFRynLoopDefaultMode：App的默认Mode,通常主线程是在这个Mode下运行  
UITrackingRunLoopMode：界面跟踪Mode,用于ScrollView追踪触摸滑动，保证界面滑动时不受其他Mode影响  
kCFRunLoopCommonModes：这是一个占位用的Mode，不是一种真正的Mode  
UIInitializationRunLoopMode：在刚启动App时进入的第一个Mode，启动完成后不再使用  
GSEventReceiveRunLoopMode：接受系统事件的内部Mode，通常用不到





应用场景举例：主线程的 RunLoop 里有两个预置的 Mode：kCFRunLoopDefaultMode 和 UITrackingRunLoopMode。这两个 Mode 都已经被标记为”Common”属性。DefaultMode 是 App 平时所处的状态，TrackingRunLoopMode 是追踪 ScrollView 滑动时的状态。当你创建一个 Timer 并加到 DefaultMode 时，Timer 会得到重复回调，但此时滑动一个TableView时，RunLoop 会将 mode 切换为 TrackingRunLoopMode，这时 Timer 就不会被回调，并且也不会影响到滑动操作。

有时你需要一个 Timer，在两个 Mode 中都能得到回调，一种办法就是将这个 Timer 分别加入这两个 Mode。还有一种方式，就是将 Timer 加入到顶层的 RunLoop 的 “commonModeItems” 中。”commonModeItems” 被 RunLoop 自动更新到所有具有”Common”属性的 Mode 里去。

