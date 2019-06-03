《今日头条iOS端安装包大小优化》[https://techblog.toutiao.com/2018/06/04/gan-huo-jin-ri-tou-tiao-iosduan-an-zhuang-bao-da-xiao-you-hua-si-lu-yu-shi-jian/](https://techblog.toutiao.com/2018/06/04/gan-huo-jin-ri-tou-tiao-iosduan-an-zhuang-bao-da-xiao-you-hua-si-lu-yu-shi-jian/)

##代码
### 1 link map
是编译链接时可以生成的一个txt文件，它生成目的就是帮助程序员分析包大小。link map记录了每个方法在当前的二进制架构下占据的空间。通过分析link map，我们可以了解每个类甚至每个方法占据了多少安装包空间。

### 2 二进制文件优化
使用技术手段排查删减冗余代码、监控代码的增长情况和分布。另外优化编译选项也是行之有效的方法

#### 2.1 排查无用类
MachO文件中有DATA.objcclassrefs和DATA.objcselrefs段，分别近似于“被使用的类的集合”和“被使用的方法的集合”。使用otool命令可查看DATA.objcclassrefs段和DATA.objcclasslist段，两者的差集可以认为是定义了但未使用的类。

不过DATA.objcclassrefs段和DATA.objcclasslist段中都只提供了类在二进制文件中的位置地址，而没有提供类名等可读信息。所以在获取到差集后，还需要结合`otool -o BinaryName`命令的输出，将地址转换成可读的类名

#### 2.2 排查无用方法
所有已经被实现的方法可以通过linkmap来获取，对linkmap做grep操作即可获得结果：
`grep '[+|-]\[.*\s.*\]'`；而所有已经被使用的方法可以通过对二进制文件逆向获得。使用otool工具逆向二进制文件的DATA.objc_selrefs 段，提取可执行文件里引用到的方法名：`otool -v -s __DATA__objc_selrefs`， 使用这种方法取到的差集，还需要排除掉系统API中的protocol，accessor方法等。

#### 2.3 extension代码精简
删除 extension 中使用较少功能的库，自己用简单方法或者系统方法实现

#### 2.4 编译选项改进
打开LTO，即Link Time Optimization
LTO能带来的优化有： （1）将一些函数內联化 （2）去除了一些无用代码 （3）对程序有全局的优化作用
**LTO会降低编译链接的速度，因此只建议在打正式包时开启**

##资源

* 排查和清除冗余图片[地址](https://github.com/tinymind/LSUnusedResources)
* ImageOptim基本可以压缩到原图片的30%
* 修改optimization设置
* 使用webP替代png
* 将图片放置到云端
* 资源文件云端下载
* 使用 icon-font 代替图片
* 利用tint color精简单色图标
* 优化 cocopods 中重复的图片


![](http://upload-images.jianshu.io/upload_images/11994763-36d37b54a1cdd29c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/11994763-5b24f5ac5b6600a2?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


