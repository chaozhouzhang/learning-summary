# Android操作系统启动流程


这是一篇译文，原英文地址请见：
```
https://learnlinuxconcepts.blogspot.com/2014/02/android-boot-sequence.html?m=0
```
在这篇文章中，我们将讨论Android操作系统的启动过程。由于Android是基于Linux内核的，所以看完它的引导过程也会对Linux的引导过程有很好的了解。大多数基于android的系统运行在ARM处理器上。

首先，我们将看到在android启动流程中使用的各种术语的含义。

|概念|解释|
|----|----|
|BootROM|它是一个驻留在CPU专用集成电路的硬连线代码。|
|Bootloader|引导加载程序，它是一个小程序，在Android操作系统开始运行之前运行。|
|Init进程|它是Linux内核完成安装后启动的第一个进程。|

让我们看看当用户按下电源开关按钮时会发生什么：
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android%20Boot%20Squence.png?raw=true)
当我们按下android手机上的电源键时，启动过程就完成了。

## 步骤1、系统启动

启动ROM代码从预先定义的位置开始执行。它将引导加载程序加载到RAM中并开始执行。

引导ROM代码将使用映射到ASIC上的一些物理球的系统寄存器来检测引导媒体。这是为了确定在何处找到引导加载程序的第一阶段。

一旦引导媒体序列建立，引导ROM将尝试加载第一阶段的引导加载程序到内部RAM。一旦引导加载程序就位，引导ROM代码将执行跳转并继续在引导加载程序中执行。

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android_boot_1.png?raw=true)
## 步骤2、引导装载程序

Bootloader的执行分为两个阶段，第一个阶段是检测外部RAM，第二个阶段是帮助加载程序，在第二个阶段Bootloader设置网络、内存等需要运行内核的时候，Bootloader可以为内核提供特定目的的配置参数或输入。

Android引导加载程序可以在以下位置发现：
```
<AndroidSource>\bootable\bootloader\legacy\usbloader
```

遗留加载程序包含两个重要的文件，即init.S和main.c
1. init.S：初始化堆栈，将BSS段归零，调用main.c中的_main()方法。
2. main.c：初始化硬件(时钟、板、键盘、控制台)，创建Linux标记。

在嵌入式Linux中，uBoot通常是选择的引导加载程序。设备制造商通常使用他们自己专有的引导加载程序。

1、第一个引导加载程序阶段将检测并设置外部RAM。

2、一旦外部RAM可用，系统准备好运行一些更重要的东西，第一阶段将加载主引导加载程序，并将其放在外部RAM中。

3、引导加载程序的第二阶段是将要运行的第一个主程序。这可能包含用于设置文件系统、额外内存、网络支持和其他内容的代码。在移动电话上，它还可能负责为调制解调器CPU加载代码，并设置低级内存保护和安全选项。

4、一旦引导加载程序完成了任何特殊任务，它就会寻找要引导的Linux内核。它将从引导媒体(或其他取决于系统配置的源)加载此文件，并将其放在RAM中。它还将在内存中放置一些引导参数，以便内核在启动时读取。

5、完成引导加载程序后，它将执行跳转到Linux内核，通常是一些解压缩例程，内核承担系统责任。

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android_boot_2.png?raw=true)

## 步骤3、Linux内核

Linux内核在Android上启动的方式与在其他系统上类似。它将设置系统运行所需的一切。初始化中断控制器，设置内存保护，缓存和调度。

一旦内存管理单元和缓存被初始化，系统将能够使用虚拟内存并启动用户空间进程。
内核将在根文件系统中查找init进程(在Android开源树的system/core/init下找到)，并将其作为初始用户空间进程启动。

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android_boot_3.png?raw=true)

## 步骤4、init进程

初始化它的第一个进程，我们可以说它是所有进程的根进程或祖母进程。
init进程有两个职责：
1、挂载目录如/sys、/dev、/proc
2、运行init.rc脚本。



init进程可以在以下位置找到: 
```
<android source>/system/core/init
```
init.rc文件可以在源代码树中找到：
```
<android source>/system/core/rootdir/init.rc
```
这是一个描述系统服务、文件系统和其他需要设置的参数的脚本。init.rc脚本被放置在Android开源项目的system/core/rootdir中。

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android_boot_4.png?raw=true)

## 步骤5、Zygote和Dalvik

在Java中，我们知道单独的虚拟机(VMs)实例会在内存中为单独的应用程序弹出，在Android应用程序应该尽快启动的情况下，如果Android os为每个应用程序启动不同的Dalvik VM实例，那么它会消耗大量的内存和时间。所以，为了解决这个问题，Android操作系统被命名为"Zygote"。

Zygote支持跨Dalvik虚拟机的共享代码，更低的内存占用和最小的启动时间。

Zygote是一个VM进程，它在系统启动时启动，我们在前面的步骤中已经知道了。

Zygote预加载并初始化核心库类。

通常核心类是只读的，是Android SDK或核心框架的一部分。在Java VM中，每个实例都有自己的核心库类文件和堆对象的副本。

简而言之，Zygote是由init进程启动的，基本上只是开始执行并初始化Dalvik VM。
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android_boot_5.png?raw=true)

## 步骤6、系统服务

系统服务是在系统中运行的第一个java组件。它将启动所有的Android服务，如电话管理器和蓝牙。

当前，每个服务的启动都直接写入系统服务的run方法中。

可以在开放源码项目的文件框架中找到系统服务：
```
/base/services/java/com/android/server/SystemServer.java
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android_boot_6.png?raw=true)

核心服务:
1.启动电源管理器
2.创建活动管理器
3.开始电话注册
4.开始包管理器
5.将活动管理器服务设置为系统流程
6.开始上下文管理器
7.启动系统上下文提供程序
8.开始电池服务
9.开始报警管理
10.开始传感器服务
11.开始窗口管理器
12.开始蓝牙服务
13.开始安装服务

其他服务：
1.启动状态栏服务
2.开始硬件服务
3.开始NetStat服务
4.开始连接服务
5.开始通知经理
6.开始DeviceStorageMonitor服务
7.开始位置管理器
8.开始搜索服务
9.开始剪贴板服务
10.开始签入服务
11.开始壁纸服务
12.开始音频服务
13.开始HeadsetObserver
14.开始AdbSettingsObserver

## 步骤7、启动完成

一旦系统服务在内存中启动并运行，Android就完成了启动过程，此时“ACTION_BOOT_COMPLETED”标准的广播动作就会触发。


欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！

![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)

