# JDK的可视化工具

## 1、JConsole Java监视与管理控制台
```
JConsole 
Java Monitoring and Management Console
基于JMX的可视化监视和管理工具，
```
```
内存监控，相当于jstat命令，用于监视受收集器管理的虚拟机内存Java堆和永久代的变化趋势。
```
```
线程监控，相当于jstack命令，监控分析线程停顿的情况。

线程停顿主要原因：等待外部资源例如数据库连接、网络资源、设备资源等；死循环；锁等待（活锁或死锁）。
```

## 2、VisualVM 多合一故障处理工具

All-in-One Java Troubleshooting Tool

```
显示虚拟机进程以及进程的配置和环境信息。jps、jinfo。
监视应用程序的CPU、GC、堆、方法区以及线程的信息。jstack、jstat。
dump以及分析堆转储快照。jmap、jhat。
方法级的程序运行性能分析，找出被调用最多、运行时间最长的方法。
离线程序快照，收集程序的运行时配置、线程dump、内存dump等信息建立一个快照，发送给开发者进行Bug反馈。
```


```
生成和浏览堆转储快照。
分析程序性能。
BTrace动态日志跟踪。
```


