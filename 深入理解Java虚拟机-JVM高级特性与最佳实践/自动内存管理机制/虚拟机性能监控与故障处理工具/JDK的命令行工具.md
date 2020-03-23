# JDK的命令行工具

|数据|
|----|
|运行日志、异常堆栈、GC日志、线程快照(threaddump/javacore文件)、堆转储快照(headdump/hprof文件)|


|工具名称|主要作用|
|----|----|
|jps|jvm process status tool,显示指定系统内所有的hotspot虚拟机进程|
|jstat|jvm statistics monitoring tool,用于收集hotspot虚拟机各方面的运行数据|
|jinfo|configuration info for java，显示虚拟机配置信息|
|jmap|memory map for java,生成虚拟机的内存转储快照（heapdump文件）|
|jhat|jvm heap dump browser，用于分析heapmap文件，它会建立一个http/html服务器让用户可以在浏览器上查看分析结果|
|jstack|stack trace for java ,显示虚拟机的线程快照|

## 1、jps 虚拟机进程状况工具
JVM Process Status Tool

LVMID
```
本地虚拟机的唯一ID，Local Virtual Machine Identifier。
```
PID
```
进程ID，Process Identifier。
```
```
可以列出正在运行的虚拟机进程，并显示虚拟机执行主类的名称，以及这些进程的本地虚拟机的唯一ID。

对于本地虚拟机进程来说，LVMID与操作系统的进程ID是一致的。

如果同时启动了多个虚拟机进程，无法根据进程名称定位时，只能依赖jps命令显示主类的功能进行区分。

jps可以通过RMI协议查询开启了RMI服务的远程虚拟机的进程状态，hostid为RMI注册表中注册的主机名。
```

|选项|作用|结果|
|----|----|----|
|-q|只输出LVMID,省略主类的名称|1270|
|-m|输出虚拟机进程启动时传递给主类main()函数的参数|1272 Jps -m|
|-l|输出主类的全名，如果进程执行的是Jar包，输出jar路径|1264 sun.tools.jps.Jps|
|-v|输出虚拟机进程启动时JVM参数|1273 Jps -Dapplication.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home -Xms8m|


## 2、jstat 虚拟机统计信息监视工具

JVM Statistics Monitoring Tool
```
监视虚拟机各种运行状态信息。

显示本地或远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
```


|选项|作用|
|----|----|
|jstat -class pid|显示加载class的数量，及所占空间等信息。|
|jstat -compiler pid|显示VM实时编译的数量等信息。|
|jstat -gc pid|可以显示gc的信息，查看gc的次数，及时间。其中最后五项，分别是young gc的次数，young gc的时间，full gc的次数，full gc的时间，gc的总时间。|
|jstat -gccapacity|可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小，如：PGCMN显示的是最小perm的内存使用量，PGCMX显示的是perm的内存最大使用量，PGC是当前新生成的perm内存占用量，PC是但前perm内存占用量。其他的可以根据这个类推， OC是old内纯的占用量。|
|jstat -gcnew pid|new对象的信息。|
|jstat -gcnewcapacity pid|new对象的信息及其占用量。|
|jstat -gcold pid|old对象的信息。|
|jstat -gcoldcapacity pid|old对象的信息及其占用量。|
|jstat -gcpermcapacity pid|perm对象的信息及其占用量。|
|jstat -util pid|统计gc信息统计。|
|jstat -printcompilation pid|当前VM执行的信息。|

## 3、jinfo Java配置信息工具
Configuration Info for Java
```
实时地查看和调整虚拟机的各项参数。
```

## 4、jmap Java内存映像工具

Memory Map for Java

```
用于生成堆转储快照headdump文件，还可以查询finalize执行队列，Java堆和永久代详细信息，如空间使用率、当前用的是哪种收集器等。
```

|选项|作用|
|----|----|
|-heap|显示java堆详细信息，如使用哪种回收器、参数配置、分代情况等。|
|-histo[:live]|显示堆中对象统计信息，包括类、实例数量、合计容量|
|-clstats|-clstats是-permstat的替代方案，显示类装载器状态|
|-finalizerinfo|显示在F-Queue中等待Finalizer线程执行finalize方法的对象|
|-dump:<dump-options>|生成java堆快照，<ump-options>在help命令下查看格式|
|-F|强制生成快照|
|-h/-help|帮助命令|
|-J<flag>|指定传递给运行jmap的JVM的参数|


## 5、jhat 虚拟机堆转储快照分析工具
JVM Heap Analysis Tool

```
Sun JDK 提供jhat和jmap搭配使用，分析jmap生成的堆转储快照。

其他工具例如VisialVM、Eclipse Memory Analyzer、IBM HeapAnalyzer等。
```


## 6、jstack Java堆栈跟踪工具
Stack Trace for Java

```
用于生成虚拟机当前时刻的线程快照theaddump或javacore文件。
```



```
线程快照是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的常见原因。
```

|选项|作用|
|----|----|
|-F|当正常输出请求不被响应时，强制输出线程堆栈|
|-l|除堆栈外，显示关于锁的附加信息|
|-m|如果调用到本地方法的话，可以显示C/C++的堆栈|



```
使用Thread类的getAllStackTraces()方法获取虚拟机中所有线程的StackTraceElement对象。
```

