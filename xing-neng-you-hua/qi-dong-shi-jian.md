[地址](https://techblog.toutiao.com/2017/01/17/iosspeed/)

# 技术调研

先说结论，t(App总启动时间) = t1(main()之前的加载时间) + t2(main()之后的加载时间)。

t1 = 系统dylib(动态链接库)和自身App可执行文件的加载； 
t2 = main方法执行之后到AppDelegate类中的- (BOOL)Application:(UIApplication *)Application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions方法执行结束前这段时间，主要是构建第一个界面，并完成渲染展示。



## main()调用之前的加载过程

App开始启动后，系统首先加载可执行文件（自身App的所有.o文件的集合），然后加载动态链接库dyld，dyld是一个专门用来加载动态链接库的库。 执行从dyld开始，dyld从可执行文件的依赖开始, 递归加载所有的依赖动态链接库。 
动态链接库包括：iOS 中用到的所有系统 framework，加载OC runtime方法的libobjc，系统级别的libSystem，例如libdispatch(GCD)和libsystem_blocks (Block)。



总结一下：对于main()调用之前的耗时我们可以优化的点有：

1. 减少不必要的framework，因为动态链接比较耗时
2. check framework应当设为optional和required，如果该framework在当前App支持的所有iOS系统版本都存在，那么就设为required，否则就设为optional，因为optional会有些额外的检查
3. 合并或者删减一些OC类，关于清理项目中没用到的类，使用工具AppCode代码检查功能，查到当前项目中没有用到的类如下：

* 删减一些无用的静态变量

* 删减没有被调用到或者已经废弃的方法

* 将不必须在+load方法中做的事情延迟到+initialize中

* 尽量不要用C++虚函数(创建虚函数表有开销)



## main()调用之后的加载时间

在main()被调用之后，App的主要工作就是初始化必要的服务，显示首页内容等。而我们的优化也是围绕如何能够快速展现首页来开展。 App通常在AppDelegate类中的- (BOOL)Application:(UIApplication *)Application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions方法中创建首页需要展示的view，然后在当前runloop的末尾，主动调用CA::Transaction::commit完成视图的渲染。 

对于main()函数调用之前我们可以优化的点有：

1. 纯代码方式而不是storyboard加载首页UI。
2. 对didFinishLaunching里的函数考虑能否挖掘可以延迟加载或者懒加载，需要与各个业务方pm和rd共同check 对于一些已经下线的业务，删减冗余代码。 
   对于一些与UI展示无关的业务，如微博认证过期检查、图片最大缓存空间设置等做延迟加载
3. 对实现了+load()方法的类进行分析，尽量将load里的代码延后调用。
4. 对于viewDidLoad以及viewWillAppear方法中尽量去尝试少做，晚做，不做。