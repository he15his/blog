# 1.内存大小优化

使用Instruments检测内存的突增，检测退出对应功能后内存是否能恢复到原值

## 图片

不会重复用到的图片使用`imageWithContentsOfFile`代替`imageNamed`能减少内存占用

也可以自己写一个弱引用的字典来存图片, value存成block；当没有地方用的这个图片的时候就会自动释放了



YYImage 的核心就是学习imageWithContentsOfFile:的方法原理去实现imageNamed:方法. 达到imageNamed:方法中没有缓存功能, 最终使得不需要图片的时候即可销毁图片对象



```objective-c
+ (YYImage *)imageNamed:(NSString *)name {

    if (name.length == 0) return nil;

    if ([name hasSuffix:@"/"]) return nil;

    NSString *res = name.stringByDeletingPathExtension;

    NSString *ext = name.pathExtension;

    NSString *path = nil;

    CGFloat scale = 1;

    // If no extension, guess by system supported (same as UIImage).

    NSArray *exts = ext.length > 0 ? @[ext] : @[@"", @"png", @"jpeg", @"jpg", @"gif", @"webp", @"apng"];

    NSArray *scales = [NSBundle preferredScales];

    for (int s = 0; s < scales.count; s++) {

        scale = ((NSNumber *)scales[s]).floatValue;

        NSString *scaledName = [res stringByAppendingNameScale:scale];

        for (NSString *e in exts) {

            path = [[NSBundle mainBundle] pathForResource:scaledName ofType:e];

            if (path) break;

        }

        if (path) break;

    }

    if (path.length == 0) return nil;

    NSData *data = [NSData dataWithContentsOfFile:path];

    if (data.length == 0) return nil;

    return [[self alloc] initWithData:data scale:scale];

}
```



## wkwebview



**WKWebView相比于UIWebView**

- WKWebView的内存远远没有UIWebView的开销大，没有缓存
- 拥有高达60FPS滚动刷新率及内置手势
- 支持了更多的HTML5特性
- 高效的app和web信息交换通道
- 允许JavaScript的Nitro库加载并使用,UIWebView中限制了
- 提供加载网页进度的属性（estimatedProgress）

- 将UIWebViewDelegate与UIWebView拆分成了14类与3个协议
- WKWebView使用和手机Safari浏览器一样的Nitro JavaScript引擎，相比于UIWebView的JavaScript引擎有了非常重要的性能提升
- 苹果提供的 WebKit 库包含了 WKWebView，WKWebView 采用**跨进程**方案，Nitro JS 解析器，高达 60fps 的刷新率





# 2.内存泄漏检测



[MLeaksFinder: 精准iOS 内存泄露检测工具](http://wereadteam.github.io/2016/02/22/MLeaksFinder/)



常见内存泄漏

*  NSTimer循环引用
*  block的循环引用
* 其它对象间相互引用