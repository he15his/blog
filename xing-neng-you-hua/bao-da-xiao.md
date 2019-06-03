《今日头条iOS端安装包大小优化》[https://techblog.toutiao.com/2018/06/04/gan-huo-jin-ri-tou-tiao-iosduan-an-zhuang-bao-da-xiao-you-hua-si-lu-yu-shi-jian/](https://techblog.toutiao.com/2018/06/04/gan-huo-jin-ri-tou-tiao-iosduan-an-zhuang-bao-da-xiao-you-hua-si-lu-yu-shi-jian/)

##代码
分析库大小link map是编译链接时可以生成的一个txt文件，它生成目的就是帮助程序员分析包大小。link map记录了每个方法在当前的二进制架构下占据的空间。通过分析link map，我们可以了解每个类甚至每个方法占据了多少安装包空间。


##资源