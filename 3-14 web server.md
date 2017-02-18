#Web服务器 如何工作？

##web原理

1. 首先我们客户端发送一个请求到Web服务器，请求首先是到网卡。
1. 网卡将请求交由内核空间的内核处理，其实就是拆包了，发现请求的是80端口。
1. 内核便将请求发给了在用户空间的Web服务器，Web服务器接受到请求发现客户端请求的index.html页面、
1. Web服务器便进行系统调用将请求发给内核
1. 内核发现在请求的是一页面，便调用磁盘的驱动程序，连接磁盘
1. 内核通过驱动调用磁盘取得的页面文件
1. 内核将取得的页面文件保存在自己的缓存区域中便通知Web进程或线程来取相应的页面文件
1. Web服务器通过系统调用将内核缓存中的页面文件复制到进程缓存区域中
1. Web服务器取得页面文件来响应用户，再次通过系统调用将页面文件发给内核
1. 内核进程页面文件的封装并通过网卡发送出去
1. 当报文到达网卡时通过网络响应给客户端

![11](http://ww4.sinaimg.cn/mw690/6be30686gw1f1wuy4lsxcj20h50ec400.jpg)


其中效率最高的是异步的方式，最稳定的是多进程方式，占用资源较少的是多线程的方式。

1，阻塞和非阻塞:

2，同步和异步:

  非阻塞I/O ，I/O复用，信号驱动式I/O其实都是非阻塞的，当然是针对“请求”这个阶段。非阻塞式是主动查询外设状态。I/O复用里的select，poll也是主动查询，不同的是select和poll可以同时查询多个fd（文件句柄）的状态，另外select有fd个数的限制。epoll是基于回调函数的。信号驱动式I/O则是基于信号消息的。这两个应该可以归到“被动接收消息”那一类中。最后就是伟大的AIO的出现，内核把什么事都干了，对上层应用实现了全异步，性能最好，当然复杂度也最高。好了，下面我们就来详细说一说，这几种模式。

对于数据输入而言，即等待(wait)数据输入至buffer需要时间，而从buffer复制(copy)数据至进程也需要时间    
根据等待模式不同，I/O动作可分为五种模式。
阻塞I/O
![1](http://ww2.sinaimg.cn/large/6be30686gw1f1wvcu87qgj20ku0c3gnr.jpg)
非阻塞I/O
![2](http://ww1.sinaimg.cn/mw690/6be30686gw1f1wvcq4b99j20ku0ahju0.jpg)
I/O复用（select和poll）
![3](http://ww3.sinaimg.cn/mw690/6be30686gw1f1wvcqrnbhj20ku0ahwgv.jpg)
信号（事件）驱动I/O（SIGIO）
![4](http://ww1.sinaimg.cn/mw690/6be30686gw1f1wvcrmu1ij20iq0cr0ti.jpg)
异步I/O（Posix.1的aio_系列函数）
![5](http://ww2.sinaimg.cn/mw690/6be30686gw1f1wvcsjhctj20j40d93ze.jpg)

对比：
![console](http://ww1.sinaimg.cn/mw690/6be30686gw1f1wuyfewcqj20m80ckmzf.jpg)

Apache 2.2.9之前只支持select模型，2.2.9之后支持epoll模型
Nginx 支持epoll模型

##高并发服务器

支持高并发的Web服务器
有几个基本条件：
1.基于线程，即一个进程生成多个线程，每个线程响应用户的每个请求。
2.基于事件的模型，一个进程处理多个请求，并且通过epoll机制来通知用户请求完成。
3.基于磁盘的AIO（异步I/O）

4.支持mmap内存映射，传统的web服务器，进行页面输入时，都是将磁盘的页面先输入到内核缓存中，再由内核缓存中复制一份到web服务器上，mmap机制就是让内核缓存与磁盘进行映射，web服务器，直接复制页面内容即可。不需要先把磁盘的上的页面先输入到内核缓存去。


##nginx

nginx采用了模块化、事件驱动、异步、单线程及非阻塞的架构，并大量采用了多路复用及事件通知机制。


## java


1，Servlet 是什么？
在Java里，**Servlet使你能够编写根据请求动态生成内容的服务端组件**。

类加载器通过懒加载（lazy-loading）或者预加载（eager loading）自动地把Servlet类加载到容器里。每个请求都拥有自己的线程，而一个Servlet对象可以同时为多个线程服务。当Servlet对象不再被使用时，它就会被JVM当做垃圾回收掉。

---cgi？

2，cookie and session

当客户端第一次访问web应用或者第一次使用request.getSession()获取HttpSession时，Servlet容器会创建Session，生成一个long类型的唯一ID（你可以使用session.getId()获取它）并把它保存在服务器的内存里。Servlet容器同样会在HTTP响应里设置一个Cookie，cookie的名是JSESSIONID并且cookie的值是session的唯一ID。

你现在应该已经知道所有的请求都在共享Servlet和Filter。这是Java的一个很棒的特性，它是多线程的并且不同的线程（即HTTP请求）可以使用同一个实例。否则，对每一个请求都重新创建一个实体会耗费很多的资源。

##http

HTTP 请求

一个 HTTP 请求包含三个部分：

Method-URI-Protocol/Version方法-地址-版本

Request header请求头

Entity body请求实体

	POST /servlet/default.jsp HTTP/1.1
	
	Accept: text/plain; text/html
	
	Accept-Language: en-gb
	
	Connection: Keep-Alive
	
	Host: localhost
	
	Referer: http://localhost/ch8/SendDetails.htm
	
	User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 98)
	
	Content-Length: 33
	
	Content-Type: application/x-www-form-urlencoded
	
	Accept-Encoding: gzip， deflate
	
	LastName=Franks&FirstName=Michael
	
	The Method-URI-Protocol/Version

HTTP 响应

与请求相似，HTTP 响应也由三部分组成：

Protocol-Status code-Description协议状态 描述代码

Response headers响应头

Entity body响应实体
	
	HTTP/1.1 200 OK
	
	Server: Microsoft-IIS/4.0
	
	Date: Mon， 3 Jan 1998 13:13:33 GMT
	
	Content-Type: text/html
	
	Last-Modified: Mon， 11 Jan 1998 13:23:42 GMT
	
	Content-Length: 112
	
	<html>

