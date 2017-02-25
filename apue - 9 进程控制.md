## unix高级环境编程 -9 - 进程控制 note笔记

- 每个进程都有一个非负整形的唯一进程ID.因为进程ID标识符总是唯一的,常将其用作其他标识符的一部分以保证其唯一性. 
- 有某些专用的进程:进程ID 0是调度进程,常常被曾为交换进程(swapper).该进程并不执行任何磁盘上的程序–它是内核的一部分,因此也被称为系统进程.进程ID 1通常是init进程.init进程决不会终止.它是一个普通的用户进程(与交换进程不同,它不是内核中的系统进程),但是它以超级用户特权运行.在某些unix的虚存实现中,进程ID 2是页精灵进程(pagedaemon,后会讲什么是daemon). 

###  创建新进程

	#include <unistd.h>
	
	pid_t fork(void);

- 创建的新进程叫做子进程，子进程是父进程的一个拷贝，拷贝数据段，堆和栈，而共享文本段。
-该函数被调用一次,但返回两次.两次返回的区别是子进程的返回值是0,而父进程的返回值则是新子进程的进程ID,将子进程ID返回给父进程的理由是:因为一个进程的子进程可以多余一个,所以没有一个函数使一个进程可以获得所有子进程的进程ID.fork使子进程得到返回值0的理由是:一个进程只会有一个父进程,所以子进程总是可以调用getppid以获得其父进程的进程ID(进程ID 0 总是由交换进程使用,所以一个子进程的进程ID不可能是0).
-写时复制（copy-on-write）机制：子进程刚创建，在只读的情况下和父进程共享数据段、堆和栈。如果子进程或者父进程试着修改这些数据，内核会进程这些数据的拷贝。


- vfork vfork和fork的不同点：

函数目的：vfork创建的子进程是为了让子进程执行一个新的程序
复制操作：不复制父进程的地址空间，而是直接运行在父进程的地址空间中，直到子进程调用exec或者exit
效率：所以vfork的执行效率比fork要高，因为它没有copy操作
不确定的结果：但是如果子进程修改了数据、调用函数或者没有调用exec和exit方法，则会造成不确定的结果
子进程先运行：vfork保证子进程先运行
#### 文件共享 

每一个打开的文件，涉及内核中的三种数据结构，这三种数据结构也是文件在进程间共享的基础。

- an entry in the process table: 每一个打开的文件描述符对应一个entry，entry中的内容包括文件描述符标志位（file descriptor flags）和一个指向file table entry的指针；
- file table：内核为所有打开的文件维护一个file table。每一个file table entry包括有：文件状态标志位（file status flag, such as read, write, append, sync和nonblocking）。
- v-node和i-node: 每一个开打的文件都有一个v-node结构体，包括文件类型，指向操作函数的指针。对于大部分的文件，v-node还包含一个i-node结构。i-node的内容为打开文件时从硬盘上读取的信息，包括文件所有者，文件大小，文件内容存储在磁盘上的具体位置等。


![](http://img.blog.csdn.net/20160228121529884)

### 程序执行（program execution）

fork创建新的进程 
exec初始化新的程序

- 调用exec的进程会完全被新的程序替代，从新程序的main开始执行
- process ID保持保持原来的值，因为没有创建新的进程
- exec替换了当前进程的text,data,heap,stacksegments
- 如果execlp或execvp通过路径找到的文件不是可执行文件，那么会认为该文件是shell script（脚本）文件，然后将其输入到shell中
l代表是list，需要命令行参数是独立的参数，我们将该argument以NULL结尾。v代表是vector.p表示使用filename参数和PATH环境变量来查找文件.e表示使用environment list


### 进程终止（process termination）

正常退出：三个函数exit， 

如果子进程不正常退出，则内核保证记录该进程的异常退出状态，该进程的父进程可以通过调用wait或者waitpid函数获取该子进程的异常退出状态。

如果父进程在子进程之前终止，则init进程成为该子进程的父进程。从而保证每个进程都有父进程。

如果子进程先终止（异常终止或者正常退出），内核会保存该子进程的部分信息，包括进程pid，进程终止时的状态和该进程占用的CPU时间，同时内核会清除该进程占用的内存，关闭所有已经打开的文件描述符。父进程可以通过检查该信息获取子进程的终止情况。

如果子进程先终止，而没有父进程调用waitpid获取该子进程的信息，那么这种进程被成为僵尸进程。使用ps命令可以看到僵尸进程的相关信息。

如果父进程为init进程，那么子进程异常终止并不会成为僵尸进程，因为init进程会对它的所有子进程调用wait函数获取子进程的终止状态。

kernel保存了所有中止进程的小部分信息(processID，中止状态，CPU执行时间)，在parent调用waitorwaitpid时可以获得这些信息。如果parent没有调用这些函数，已经中止的子进程处于zombie状态（使用ps可以查看有Z标志的进程）

- wait：不管一个进程有几个子进程，父进程执行wait的时候就一直在那里等，不管哪个子进程终止，wait就返回其ID； 
- waitpid：通过设置pid参数，可以等待任一子进程(-1)，或者等待指定子进程(>0)，或者等待group ID的相关进程(不多说了)等。 

		#include <sys/wait.h>
		
		pid_t wait(int *statloc);
		
		pid_t waitpid(pid_t pid, int *statloc, int options); 
		
		// Both return: process ID if OK, 0,or -1 on error
参数statloc是一个整型指针，如果该参数不为null，则子进程的终止状态被保存在该参数指向的整型中；如果我们不关心进程的终止状态，statloc传入null就行；

### 特殊的进程ID

- 可以用setuid函数设置实际用户ID和有效用户ID.与此类似,可以用setgid函数设置实际组ID和有效组ID.

-三个用户ID 
![1](http://img.blog.csdn.net/20170123225003717?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWl95aWNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)