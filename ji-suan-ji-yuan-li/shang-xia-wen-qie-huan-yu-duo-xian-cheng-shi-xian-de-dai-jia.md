# [上下文切换与多线程实现的代价](https://www.cnblogs.com/binyue/p/4484783.html)

## 多线程中的上下文切换

支持多任务处理是CPU设计史上最大的跨越之一。在计算机中，多任务处理是指同时运行两个或多个程序。从使用者的角度来看，这看起来并不复杂或者难以实现，但是它确实是计算机设计史上一次大的飞跃。在多任务处理系统中，CPU需要处理所有程序的操作，当用户来回切换它们时，需要记录这些程序执行到哪里。上下文切换就是这样一个过程，他允许CPU记录并恢复各种正在运行程序的状态，使它能够完成切换操作。

在上下文切换过程中，CPU会停止处理当前运行的程序，并保存当前程序运行的具体位置以便之后继续运行。从这个角度来看，上下文切换有点像我们同时阅读几本书，在来回切换书本的同时我们需要记住每本书当前读到的页码。在程序中，上下文切换过程中的“页码”信息是保存在进程控制块（PCB）中的。PCB还经常被称作“切换桢”（switchframe）。“页码”信息会一直保存到CPU的内存中，直到他们被再次使用。

在三种情况下可能会发生上下文切换：中断处理，多任务处理，用户态切换。在中断处理中，其他程序”打断”了当前正在运行的程序。当CPU接收到中断请求时，会在正在运行的程序和发起中断请求的程序之间进行一次上下文切换。在多任务处理中，CPU会在不同程序之间来回切换，每个程序都有相应的处理时间片，CPU在两个时间片的间隔中进行上下文切换。对于一些操作系统，当进行用户态切换时也会进行一次上下文切换，虽然这不是必须的。

操作系统或者计算机硬件都支持上下文切换。一些现代操作系统通过系统本身来控制上下文切换，整个切换过程中并不依赖于硬件的支持，这样做可以让操作系统保存更多的上下文切换信息。

## 实现多线程的代价

从一个单线程的应用到一个多线程的应用并不仅仅带来好处，它也会有一些代价。不要仅仅为了使用多线程而使用多线程。而应该明确在使用多线程时能多来的好处比所付出的代价大的时候，才使用多线程。如果存在疑问，应该尝试测量一下应用程序的性能和响应能力，而不只是猜测。

### 设计更复杂

虽然有一些多线程应用程序比单线程的应用程序要简单，但其他的一般都更复杂。在多线程访问共享数据的时候，这部分代码需要特别的注意。线程之间的交互往往非常复杂。不正确的线程同步产生的错误非常难以被发现，并且重现以修复。

### 上下文切换的开销

当CPU从执行一个线程切换到执行另外一个线程的时候，它需要先存储当前线程的本地的数据，程序指针等，然后载入另一个线程的本地数据，程序指针等，最后才开始执行。这种切换称为“上下文切换”\(“context switch”\)。CPU会在一个上下文中执行一个线程，然后切换到另外一个上下文中执行另外一个线程。

上下文切换并不廉价。如果没有必要，应该减少上下文切换的发生。

你可以通过维基百科阅读更多的关于上下文切换相关的内容：

[**http://en.wikipedia.org/wiki/Context\_switch**](http://en.wikipedia.org/wiki/Context_switch)

### 增加资源消耗

线程在运行的时候需要从计算机里面得到一些资源。除了CPU，线程还需要一些内存来维持它本地的堆栈。它也需要占用操作系统中一些资源来管理线程。我们可以尝试编写一个程序，让它创建100个线程，这些线程什么事情都不做，只是在等待，然后看看这个程序在运行的时候占用了多少内存。



## 并发编程中的竞争条件 

### 下面关于竞争条件的描述，来自《现代操作系统》

在一些操作系统中，协作的进程可能共享一些彼此都能读写的公用存储区。这个公用存储区可能在内存中（可能是在内核数据结构中），也可能是一个共享文件。这里共享存储区的位置并不影响通信的本质及其带来的问题。为了理解实际中进程间通信如何工作，我们考虑一个简单但很普遍的例子：一个假脱机打印程序。当一个进程需要打印一个文件时，它将文件名放在一个特殊的假脱机目录 （spooler directory）下。另一个进程（打印机守护进程）则周期性地检查是否有文件需要打印，若有就打印并将该文件名从目录下删掉。

设想假脱机目录中有许多槽位，编号依次为0，1，2，…，每个槽位存放一个文件名。同时假设有两个共享变量：out，指向下一个要打印的文件；in，指向目录中下一个空闲槽位。可以把这两个变量保存在一个所有进程都能访问的文件中，该文件的长度为两个字。在某一时刻，0号至3号槽位空（其中的文件已经打印完毕），4号至6号槽位被占用（其中存有排好队列的要打印的文件名）。几乎在同一时刻，进程A和进程B都决定将一个文件排队打印，这种情况如图2-21所示。

| [![](http://images.51cto.com/files/uploadimg/20090805/113745925.jpg)](http://images.51cto.com/files/uploadimg/20090805/113745925.jpg) |
| :--- |


在Murphy法则（任何可能出错的地方终将出错）生效时，可能发生以下的情况。进程A读到in的值为7，将7存在一个局部变量next\_free\_slot中。此时发生一次时钟中断，CPU认为进程A已运行了足够长的时间，决定切换到进程B。进程B也读取in，同样得到值为7，于是将7存在B的局部变量next\_free\_slot中。在这一时刻两个进程都认为下一个可用槽位是7。

进程B现在继续运行，它将其文件名存在槽位7中并将in的值更新为8。然后它离开，继续执行其他操作。

最后进程A接着从上次中断的地方再次运行。它检查变量next\_free\_slot，发现其值为7，于是将打印文件名存入7号槽位，这样就把进程B存在那里的文件名覆盖掉。然后它将next\_free\_slot加1，得到值为8，就将8存到in中。此时，假脱机目录内部是一致的，所以打印机守护进程发现不了任何错误，但进程B却永远得不到任何打印输出。类似这样的情况，**即两个或多个进程读写某些共享数据，而最后的结果取决于进程运行的精确时序，称为竞争条件（race condition）**。调试包含有竞争条件的程序是一件很头痛的事。大多数的测试运行结果都很好，但在极少数情况下会发生一些无法解释的奇怪现象。

  


