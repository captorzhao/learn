## unix高级环境编程 -8 - 进程环境 note笔记


1. 程序运行时，main函数是如何被调用的；

	Some of this comes with the runtime library's crt0.o file or its __start() function. 
	
	Depending on the system you're using the follwing may be incomplete, but it should give you an idea. Using newlib-1.9.0/libgloss/m68k/crt0.S as an outline, the steps are:
	1. Set stack pointer to value of __STACK if set 
	2. Set the initial value of the frame pointer 
	3. Clear .bss (where all the values that start at zero Go) 
	4. Call indirect of hardware_init_hook if set to initialize hardware 
	5. Call indirect of software_init_hook if set to initialize software 
	6. Add __do_global_dtors and __FINI_SECTION__ to the atexit function so destructors and other cleanup functions are called when the program exits by either returning from main, or calling exit
	7. setup the paramters for argc, argv, argp and call main 
	8. call exit if main returns	
	

	　
　　　 程序连接器会寻找。 __start 这个符号是程序的起始点。 main 是被标准库stdlib 在链接时 调用的一个符号
	不同的编译器，不一定缺省得符号都是 __start。
	
	在编译器的环境中找到一个名字类似于 crt0.o 的文件，这个文件中包含了我们刚才所说的 __start 符号。（crt 大概是 C Runtime 的缩写，请大家帮助确认一下。）


2. 命令行参数是如何被传入到程序中的；
	
	作为字符串存储在内存 argv[] 保存的指针地址

3. 一个典型的内存布局是怎样的；
	
	高地址 命令行参数，环境变量 然后 是栈，低地址是依次是静态代码，全局、静态变量，未初始化块， 堆heap地址。

	
	文本段（Text Segment），保存CPU将要执行的机器指令。文本段是可共享的，所以某个程序多次执行时，对应的文本段只需要在内存中存有一份拷贝。文本段是只读的（read-only），防止程序的指令被修改。
	已初始化数据段（initialized data segment），保存程序中被初始化的全局变量（定义在任何函数之外）。例如：int maxcount = 99; 全局变量变量maxcount被保存在初始化数据段。
	
	未初始化数据段（uninitialized data segment），也被称为BSS（block started by symbol），这个段中的数据在程序执行之前被内核初始化为0或者null。;例如定义一个全局变量（定义在任何函数之外），long sum[1000];  该变量保存在未初始化数据段中。
	
	栈（Stack）：存储临时变量，函数相关信息。当一个函数被调用时，返回地址、调用者相关信息（如寄存器信息）会被保存在栈中。该被调用的函数会在栈上分配一部分空间保存它的临时变量。函数的递归调用也是应用这个原理。每一次函数调用自己，都会保存当前函数的信息，然后再栈上开辟一个新的空间用于保存该次函数的信息，和以前的函数并没有影响。
	
	堆（Heap）：动态内存分配位置。堆的位置位于未初始化数据段和栈的中间。

4. 如何分配内存；

		#include <stdlib.h>
		
		void *malloc(size_t size);
		
		void *calloc(size_t nobj, size_t size);
		
		void *realloc(void *ptr, size_t newsize);
		
		void free(void* ptr);


	内存分配函数使用系统调用sbrk来实现。该系统调用的作用是扩展进程的堆	

5. 程序如何使用环境变量；
	
获取环境变量值使用函数getenv。

	#include <stdlib.h>
	
	char* getenv(const char* name);
	
	// Returns: pointer to value associated with name, NULL if not found

修改环境变量的函数：

	#include <stdlib.h>
	
	int putenv(char* str);
	
	int setenv(const char* name, const char* value, int rewrite);
	
	int unsetenv(const char* name);

6. 程序终止的各种方式；
	
5种正常3种异常方式（abort，信号，最后一个线程应答）。

	#include <stdlib.h>
	
	int atexit(void (*func)(void));

atexit函数。这些退出句柄的调用顺序为注册时的相反顺序
exit函数第一次调用退出句柄时，会关闭所有打开的流
如果主程序调用了exec系列函数，则所有注册的退出句柄都会被清空

7. 跳转（longjmp和setjmp）函数的工作方式，以及如何和栈交互；

setjmp, sigsetjmp - save stack context for nonlocal goto
系统内部并有没有支持stack的硬件，C的实现可能会使用链表来实现stack frames。
Stack frame（堆栈帧）是一个为函数保留的区域，用来存储关于参数、局部变量和返回地址的信息。堆栈帧通常是在新的函数调用的时候创建，并在函数返回的时候销毁。
Call stack（调用堆栈）:调用堆栈是一个方法列表，按调用顺序保存所有在运行期被调用的方法。方法栈是JVM为对象的每一次方法调用所分配的一块独立的内存空间

	#include <setjmp.h>
	
	int setjmp(jmp_buf env);
	//Returns: 0 if called directly, nonzero if returning from a call to longjmp
	
	void longjmp(jmp_buf env, int val);

longjmp必须在setjmp调用之后，而且longjmp必须在setjmp的作用域之内。具体来说，在一个函数中使用setjmp来初始化一个全局标号，然后只要该函数未曾返回，那么在其它任何地方都可以通过longjmp调用来跳转到 setjmp的下一条语句执行。实际上setjmp函数将发生调用处的局部环境保存在了一个jmp_buf的结构当中，只要主调函数中对应的内存未曾释放 （函数返回时局部内存就失效了），那么在调用longjmp的时候就可以根据已保存的jmp_buf参数恢复到setjmp的地方执行。

8. 进程的资源限制

每个进程都有一组资源的限制，通过getrlimit、setrlimit能查寻和改变资源限制

	#include <sys/time.h>
	#include <sys/resource.h>
	
	int getrlimit(int resource, struct rlimit *rlim);
	int setrlimit(int resource, const struct rlimit *rlim);
	
	//Return： 0 if OK， nonzero on error

resource：可能的选择有

RLIMIT_AS //进程的最大虚内存空间，字节为单位。

RLIMIT_CORE //内核转存文件的最大长度。

RLIMIT_CPU //最大允许的CPU使用时间，秒为单位。当进程达到软限制，内核将给其发送SIGXCPU信号，这一信号的默认行为是终止进程的执行。然而，可以捕捉信号，处理句柄可将控制返回给主程序。如果进程继续耗费CPU时间，核心会以每秒一次的频率给其发送SIGXCPU信号，直到达到硬限制，那时将给进程发送 SIGKILL信号终止其执行。

RLIMIT_DATA //进程数据段的最大值。

RLIMIT_FSIZE //进程可建立的文件的最大长度。如果进程试图超出这一限制时，核心会给其发送SIGXFSZ信号，默认情况下将终止进程的执行。

RLIMIT_LOCKS //进程可建立的锁和租赁的最大值。

RLIMIT_MEMLOCK //进程可锁定在内存中的最大数据量，字节为单位。

RLIMIT_MSGQUEUE //进程可为POSIX消息队列分配的最大字节数。

RLIMIT_NICE //进程可通过setpriority() 或 nice()调用设置的最大完美值。

RLIMIT_NOFILE //指定比进程可打开的最大文件描述词大一的值，超出此值，将会产生EMFILE错误。

RLIMIT_NPROC //用户可拥有的最大进程数。

RLIMIT_RTPRIO //进程可通过sched_setscheduler 和 
sched_setparam设置的最大实时优先级。

RLIMIT_SIGPENDING //用户可拥有的最大挂起信号数。

RLIMIT_STACK //最大的进程堆栈，以字节为单位。

	ulimit -c -n -s -H
	-H表示显示的是hard limit