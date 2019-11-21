### openjdk
  This file should be located at the top of the hotspot Mercurial repository.

  See http://openjdk.java.net/ for more information about the OpenJDK.

  See ../README-builds.html for complete details on build machine requirements.

Simple Build Instructions:

    cd make && gnumake
     
  The files that will be imported into the jdk build will be in the "build"
  directory.

### 源码目录介绍

``` lua
src
├── cpu [CPU相关]
|    ├── ppc
|    ├── sparc
|    ├── x86
|    ├── zero
├── os [操作系统相关]
|    ├── aix
|    ├── bsd
|    ├── linux
|    ├── posix
|    ├── solaris
|    ├── windows
├── os_cpu [系统CPU相关]
|    ├── aix_ppc
|    ├── bsd_x86
|    ├── bsd_zero
|    ├── linux_ppc
|    ├── linux_sparc
|    ├── linux_x86
|    ├── linux_zero
|    ├── solaris_sparc
|    ├── solaris_x86
|    ├── windows_x86
├── share
|    ├── tools [工具]
|    ├── vm [虚拟机]
|         ├── adlc [平台描述文件]
|         ├── asm [汇编器]
|         ├── c1 [client编译器，也叫c1编译器]
|         ├── ci [动态编译器]
|         ├── classfile [class字节码文件相关代码]
|         ├── code [机器码生成]
|         ├── compiler [调用动态编译器的接口]
|         ├── gc_implementation [gc具体实现]
|               ├── concurrentMarkSweep [CMS：牺牲吞吐量为代价来获得最短回收停顿时间的垃圾回收器，算法：标记-清除]
|               ├── g1 [G1是一个分代的，增量的，并行与并发的标记-复制垃圾回收器]
|               ├── parallelScavenge [parallelScavenge用于年轻代回收的使用复制算法的并行收集器]
|               ├── parNew [ParNew收集器收集器其实就是Serial收集器的多线程版本]
|               ├── shared [垃圾回收实现公共代码]
|         ├── gc_interface [gc接口]
|         ├── interpreter [解释器]
|         ├── libadt [抽象数据结构]
|         ├── memory [内存管理]
|         ├── oops [JVM内部对象表示]
|         ├── opto [server编译器，即c2编译器]
|         ├── precompiled [预编译头文件]
|         ├── prims [Hotspot对外接口]
|         ├── runtime [运行时]
|         ├── services [JMX接口]
|         ├── shark [基于LLVM实现的即时编译器]
|         ├── trace [跟踪]
|         ├── utilities [内部工具类和公共函数]

```


