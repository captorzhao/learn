Linux后台开发必备技能

###1 . 系统工具 netstat tcpdump ipcs ipcrm 

打印TCP会话中的的开始和结束数据包, 并且数据包的源或目的不是本地网络上的主机.(nt: localnet, 实际使用时要真正替换成本地网络的名字))

	tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'
打印所有源或目的端口是80, 网络层协议为IPv4, 并且含有数据,而不是SYN,FIN以及ACK-only等不含数据的数据包.(ipv6的版本的表达式可做练习)

	tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
(nt: 可理解为, ip[2:2]表示整个ip数据包的长度, (ip[0]&0xf)<<2)表示ip数据包包头的长度(ip[0]&0xf代表包中的IHL域, 而此域的单位为32bit, 要换算成字节数需要乘以4,　即左移2.　(tcp[12]&0xf0)>>4 表示tcp头的长度, 此域的单位也是32bit,　换算成比特数为 ((tcp[12]&0xf0) >> 4)<<２,即 ((tcp[12]&0xf0)>>2).　((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0　表示: 整个ip数据包的长度减去ip头的长度,再减去tcp头的长度不为0, 这就意味着, ip数据包中确实是有数据.对于ipv6版本只需考虑ipv6头中的'Payload Length' 与 'tcp头的长度'的差值, 并且其中表达方式'ip[]'需换成'ip6[]'.)

		netstat -natp
		tcpdump tcp port 22 -XX -c 10
		tcpdump ip host 210.27.48.1 and ! 210.27.48.2
		ipcs -a -m -q -s 
		ipcs -t p c l u
		ipcrm -M m Q q S s 
		 
###2 .  cpu 内存 硬盘 等等与系统性能调试相关的命令 设置修改权限 tcp网络状态查看 各进程状态

1. 用vmstat、sar、iostat检测是否是CPU瓶颈 
2. 用free、vmstat检测是否是内存瓶颈
3. 用iostat检测是否是磁盘I/O瓶颈
4. 用netstat检测是否是网络带宽瓶颈
5. 进程维度 pmap gdb time strace、ltrace gprof.

[linux进程的几个状态]

1. Linux进程状态：R (TASK_RUNNING)，可执行状态&运行状态(在run_queue队列里的状态)

2. Linux进程状态：S (TASK_INTERRUPTIBLE)，可中断的睡眠状态, 可处理signal

3. Linux进程状态：D (TASK_UNINTERRUPTIBLE)，不可中断的睡眠状态,　可处理signal,　有延迟

4. Linux进程状态：T (TASK_STOPPED or TASK_TRACED)，暂停状态或跟踪状态,　不可处理signal,　因为根本没有时间片运行代码

5. Linux进程状态：Z (TASK_DEAD - EXIT_ZOMBIE)，退出状态，进程成为僵尸进程。不可被kill,　即不响应任务信号,　无法用SIGKILL杀死

###3.  grep awk sed
sed -n '1000 , 1005p' log.log

###4. 共享内存的使用实现原理、然后共享内存段被映射进进程空间之后，存在于进程空间的什么位置？共享内存段最大限制是多少？

nmap函数要求内核创建一个新额虚拟存储器区域，最好是从地质start开始的一个区域，并将文件描述符fd指定对象的一个连续的片（chunk）映射到这个新的区域。

 SHMMNI为128，表示系统中最多可以有128个共享内存对象。

###5. c++进程内存空间分布

![](http://images.cnblogs.com/cnblogs_com/skynet/201103/201103071829123774.png)

###6. ELF是什么？其大小与程序中全局变量的是否初始化有什么关系（注意.bss段）

Linux ELF ELF = Executable and Linkable Format，可执行连接格式 作为应用程序二进制接口（Application Binary Interface，ABI）

.data  初始化了的全局静态变量和局部静态变量 ; .bss未初始化的全局变量和局部静态变量 ; .rodata  只读数据(字符串常量)变量bss_array的大小为4M，而可执行文件的大小只有5K。 由此可见，bss类型的全局变量只占运行时的内存空间，而不占文件空间。仅仅是把初始化的值改为非零了，文件就变为4M多。由此可见，data类型的全局变量是即占文件空间，又占用运行时内存空间的。

###7 进程间通信

共享内存， socket，pipe，信号。 ipcs


###8. makefile编写
