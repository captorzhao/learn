## 9 进程关系

### 1. 终端登录（Terminal Logins）

exec 不会改变进程ID 只会用新的程序代替现有程序开始执行。

程序getty的职责：为终端设备调用open函数，一旦设备被打开，文件描述符0，1，2被设置给该设备。然后getty输出一些提示符，等待我们输入用户名。当我们输入用户名后，getty的工作就完成了，然后通过调用exec函数执行登录函数

![](http://images.cnitblog.com/blog/526303/201504/022122113894677.png)

### 2 网络登录

为了统一处理物理登录和网络登录，一个软件驱动，叫做虚拟终端（pseudo terminal）被用来用将网络登录后的行为请求映射为真实终端的行为。

1. 系统启动时，init进程创建一个shell执行脚本/etc/rc，其中一个后台进程就是inetd。一旦该脚本终止，inetd进程的父进程就成为了init进程；
2. inetd的职责是等待TCP/IP连接请求，一旦有新的连接请求到来，inetd会执行fork and exec执行相应的处理程序；
3. telnetd程序会启动一个TELNET服务器，等待用户远程登录，用户通过TCP协议链接服务器，并通过合法的用户密码进行登录。

![](http://images.cnitblog.com/blog/526303/201504/022122154986189.png)

![](http://img.blog.csdn.net/20170124222102653?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWl95aWNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 3 进程组

进程组是一些进程的集合，这些进程常常关联于同一个job，并且从同一个终端接收信号。

每一个进程组都有一个唯一的进程组ID。
函数getpgrp返回调用进程的进程组ID。

	#include <unistd.h>
	
	pid_t getpgrp(void);
	
	        // Returns: process group ID of calling process
	
	 
	
	pid_t getpgid(pid_t pid);
	
	        // Returns: process group ID if OK, -1 on error


### 4 Sessions 会话

一个session是一个或几个进程组的集合。

### 5， 作业控制， 前台作业， 后台作业 进程组。

用fg命令 将作业送至前台作业组。
stty 控制

进程的终端控制， /dev/tty

### 6，orphaned 孤儿进程组

