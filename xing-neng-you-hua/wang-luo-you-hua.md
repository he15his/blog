[链接](http://www.mrpeak.cn/blog/ios-network/)



### 优化清单

##### DNS映射

无论是HTTP还是Socket长连接，第一步都是DNS解析。域名根据层级「主机名.次级域名.顶级域名.根域名」去解析，每一级缓存生命周期不同。在iOS设备上几乎每次断网重连，重启设备都会使DNS缓存失效，触发重新查询。这一步的优化对请求的延迟来说至关重要，具体优化手段可参考我之前一篇关于[DNS映射的文章](http://www.mrpeak.cn/ios/2016/01/22/dnsmapping)，配有可用的demo代码，这里就不复述了。



可以使用腾讯的 [HPPTDNS](https://github.com/tencentyun/httpdns-ios-sdk)

##### 请求压缩

DNS查询之后是TCP握手建立连接，并发送请求数据。对于TCP来说，单个IP包大小受限于MSS值，大部分用户所处网络环境下每个包的大小约在1.5KB，新建立的HTTP连接由于TCP的slow start特性，会导致本地的部分IP包本临时缓存，从而增加了整体request的延迟。所以我们应该尽可能尝试去压缩我们的网络请求业务数据，减少一个Request的IP包数量，或许可以让用户少经历一个RTT，降低请求延迟的用户感知。

##### 请求合并

对于非关键性的业务数据，或者对实时性要求不高的请求来说，通过合并请求的方式可以减少和服务器交互的次数，一则降低服务器压力，二则合并之后再压缩能节约客户端的流量。这类请求一般见于打点SDK，crash日志收集等非业务型请求。

##### 请求的安全性

请求的网络安全是个容易被忽视的话题，关于安全我之前也写过一篇比较[详细的文章](http://www.mrpeak.cn/blog/encrypt/)，建议细读再配合使用HTTPS来做到基本的网络安全，这里也不再细述了。

##### 合理的并发数

有些业务场景会出现多个Request集中产生的情况，此时我们需要设置一个合理的并发数。并发数如果太小，会导致“劣质”的请求block住“优质”的请求。如果并发数太大，带宽有限的场景下，会增加请求的整体延迟，请求数量对于HTTP的影响我在[之前的文章](http://www.mrpeak.cn/blog/http2/)中也详细的介绍过了。

##### 可靠性保障

可靠性保障也是个容易被忽视的方面，在深入探讨之前，可以先将Request按业务属性分类。

- 第一类：关键核心的业务数据，期望能100%送达服务器。
- 第二类：重要内容请求，需要较高的请求成功率。
- 第三类：一般性内容请求，对成功率无要求。

##### 多通道

现在不少有技术条件的团队都有自己的tcp长连接通道，技术再硬点的甚至配有UDP通道，UDP在丢包率高的网络环境下能极大的提高请求成功的概率。如果能同时具备HTTP，TCP，UDP三条网络通道，在某些场景下，如果不考虑流量（比如wifi），可以针对某个网络请求，两通道或者三通道齐发，对请求成功的速度和可靠性有明显的疗效，不过客户端和服务器都需要针对业务场景做去重。我工作过的一个IM App在发送消息的时候，就是Socket配合HTTP双通道工作。UDP在VOIP服务当中使用较多，不过据说淘宝这类大厂也部分启用了UDP。

##### 网络环境监控

现在网络环境虽然越来越好，Wifi，4G，3G在一二线城市都有很好的普及，但还是有不少场景会导致网络状态突然变差，比如进电梯，做火车，人多的集会场所，从公司回家4G切Wifi等等，这些场景在生活当中并不少见，健壮的网络模块需要仔细的检测网络的变化，针对性的做请求重试。

##### 请求成功率监控

网络模块应该能监控当前App的网络请求成功率，对于失败率较高的请求，带上业务数据，手机网络环境，系统参数等等，在用户不活跃的时候能打包上报给server端，一则能找出更多需要优化的业务场景，二则能实时监控server端的健康状态，三则能从数据层面精确判断每一次网络优化是否有成效。