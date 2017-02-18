## 从编译到进程基础

## 调试性能分析工具

 - 程序 
 elf 文件
readelf objdump
 strings

链接 ： 动态链接， 
PIC 地址无关代码
GOT  数组
PLT&延迟绑定。 过程连接表，代码段中。二次跳转，重定位之后放到GOT中
nm
符号解析
strip 
ldd

#### 进程

- pmap  进程的内存空间
- maps 
- gdb 
- time
- fork 下 /proc/*/stat |awk '{print $14,$15}'
- strace 
- ltrace
- gprof
- pref top

#### 系统

- vmstat 负载cpu mem


2. 