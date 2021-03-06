一般的区别：
* 1.post更安全（不会作为url的一部分，不会被缓存、保存在服务器日志、以及浏览器浏览记录中）
* 2.post发送的数据量更大（get有url长度限制）
* 3.post能发送更多的数据类型（get只能发送ASCII字符）
* 4.post比get慢



我相信不止一个人跟我一样有这种疑惑，既然post有这么多优点，那我们为什么要使用get？甚至有个同事说，咱们封装一个ajax底层，直接不用get算了……

　　但是，get比post更快，那究竟快多少呢？表现在哪些方面？

 **1.post请求包含更多的请求头**

　　因为post需要在请求的body部分包含数据，所以会多了几个数据描述部分的首部字段（如content-type），这其实是微乎其微的

**2.最重要的一条，post在真正接受数据之前会先将请求头发送给服务器进行确认，然后才真正发送数据**

　　**post请求的过程：**

　　1.浏览器请求tcp连接（第一次握手）

　　2.服务器答应进行tcp连接（第二次握手）

　　3.浏览器确认，并发送post请求头（第三次握手，这个报文比较小，所以http会在此时进行第一次数据发送）

　　4.服务器返回100 continue响应

　　5.浏览器开始发送数据

　　6.服务器返回200 ok响应

 

　　**get请求的过程**

　   1.浏览器请求tcp连接（第一次握手）

　　2.服务器答应进行tcp连接（第二次握手）

　　3.浏览器确认，并发送get请求头和数据（第三次握手，这个报文比较小，所以http会在此时进行第一次数据发送）

　　4.服务器返回200 ok响应

 

　　也就是说，目测get的总耗是post的2/3左右

　　口说无凭，已经有网友进行测试了

　　[寻根究底:Ajax请求的GET与POST方式比较](http://www.oncoding.cn/2009/ajax-get-post/)

**3.get会将数据缓存起来，而post不会**

　　可以做个简短的测试，使用ajax采用get方式请求静态数据（比如html页面，图片）的时候，如果两次传输的数据相同，第二次以后耗费的时间将在10ms以内（chrome测试），而post每次耗费的时间都差不多……

　　经测试，chrome下和firefox下如果检测到get请求的是静态资源，则会缓存，如果是数据，则不缓存，但是IE这个傻X啥都会缓存起来

　　当然，应该没人会用post去获取静态数据吧，反正我是没看到过。

**4.post不能进行管道化传输**

　　http权威指南中是这样说的：

　　http在的一次会话需要先建立tcp连接（大部分是tcp，但是其他安全协议也是可以的），然后才能通信，如果每次连接都只进行一次http会话，那这个连接过程占的比例太大了！

　　于是出现了持久连接：在http/1.0+中是connection首部中添加keep-alive值，在http/1.1中是在connection首部中添加persistent值，当然两者不仅仅是命名上的差别，http/1.1中，持久连接是默认的，除非显示在connection中添加close，否则持久连接不会关闭，而http/1.0+中则恰好相反，除非显示在connection首部中添加keep-alive，否则在接收数据包后连接就断开了。

　　出现了持久连接还不够，在http/1.1中，还有一种称为管道通信的方式进行速度优化：把需要发送到服务器上的所有请求放到输出队列中，在第一个请求发送出去后，不等到收到服务器的应答，第二个请求紧接着就发送出去，但是这样的方式有一个问题：**不安全，如果一个管道中有10个连接，在发送出9个后，突然服务器告诉你，连接关闭了，此时客户端即使收到了前9个请求的答复，也会将这9个请求的内容清空，也就是说，白忙活了……此时，客户端的这9个请求需要重新发送。这对于幂等请求还好（比如get，多发送几次都没关系，每次都是相同的结果），如果是post这样的非幂等请求（比如支付的时候，多发送几次就惨了），肯定是行不通的。**

　　**所以，post请求不能通过管道的方式进行通信！**

　　**很有可能，post请求需要重新建立连接，这个过程不跟完全没优化的时候一样了么？**

　　所以，在可以使用get请求通信的时候，不要使用post请求，这样用户体验会更好，当然，如果有安全性要求的话，post会更好。