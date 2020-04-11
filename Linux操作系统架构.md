# Linux操作系统架构


|架构组成|解释|
|----|----|
|Computer Resources|计算机硬件资源|
|Kernel|操作系统内核|
|Shell|系统的用户界面，提供用户与内核进行交互操作的接口，接收用户输入的命令并把命令送入内核去执行，是一个命令解释器|
|Programs/Utilities/Tools|库函数、工具等|
|File systems|文件系统是文件存放在磁盘等存储设备上的组织方法。Linux系统能支持多种目前流行的文件系统，如EXT2、 EXT3、 FAT、 FAT32、 VFAT和ISO9660。|
|User Application|Linux应用，标准的Linux系统一般都有一套被称为应用程序的程序集，它包括文本编辑器、编程语言、X Window、办公套件、Internet工具和数据库等|

## Linux操作系统启动流程
1、Linux开机后，内核启动，激活内核空间，抽象硬件、初始化硬件参数等，运行并维护虚拟内存、调度器、信号及进程间通信(IPC)。

2、内核启动后，再加载Shell和用户应用程序，用户应用程序使用C\C++编写，被编译成机器码，形成一个进程，通过系统调用（Syscall）与内核系统进行联通。进程间交流需要使用特殊的进程间通信(IPC)机制。


# Android操作系统
## 1、应用层（System Apps）
该层中包含所有的Android应用程序，包括电话、相机、日历等，我们自己开发的Android应用程序也被安装在这层；大部分的应用使用JAVA开发，现在Google也开始力推kotlin进行开发

## 2、应用框架层（Java API Framework）
这一层主要提供构建应用程序是可能用到的各种API，Android自带的一些核心应用就是使用这些API完成的，开发者也可以通过使用API来构建自己的应用程序。

## 3、运行层
### 3.1、系统Native库
Android包含一些C/C++库，这些库能被Android系统中不同的组件使用
### 3.2、Android运行时环境
Android包括了一个核心库，该核心库提供了Java编程语言核心库的大多数功能。虚拟机也在该层启动。
每个Android应用都有一个专有的进程，这些进程每个都有一个Dalivik虚拟机实例，并在该实例中运行。

## 4、硬件抽象层(HAL)
Android的硬件驱动与Linux不同，传统的Liunx内核驱动完全存在于内核空间中。但是Android在内核外部增加了一个硬件抽象层(HAL-Hardware Abstraction Layer)，把一部分硬件驱动放到了HAL层。

### 4.1、为什么Android要这么做呢？
Linux内核采用了GPL协议，如果硬件厂商需要支持Linux系统，就需要遵照GPL协议公开硬件驱动的源代码，这势必会影响到硬件厂家的核心利益。

Android的HAL层运行在用户空间，HAL是一个"空壳"，Android会根据不同的需要，加载不同的动态库。这些动态库由硬件厂家提供。硬件厂家把相关硬件功能写入动态库，内核中只开放一些基本的读写接口操作。这样一些硬件厂家的驱动功能就由内核空间移动到了用户空间。

Android的HAL层遵循Apache协议，并不要求它的配套程序，因此厂家提供的驱动库不需要进行开放，保护了硬件厂家的核心利益。

## 5、Liunx 内核(Kernel)

Android平台的基础是Linux内核，比如ART虚拟机最终调用底层Linux内核来执行功能。Linux内核的安全机制为Android提供相应的保障，也允许设备制造商为内核开发硬件驱动程序。

# Android操作系统启动流程



### 1、BootRom BootLoader
手机开机后，引导芯片启动，引导芯片开始从固化在ROM里的预设代码执行，加载引导程序到RAM，bootloader检查RAM，初始化硬件参数等功能；

### 2、Kernel
硬件等参数初始化完成后，进入到Kernel层，Kernel层主要加载一些硬件设备驱动，初始化进程管理等操作。在Kernel中首先启动swapper进程（pid=0），用于初始化进程管理、内管管理、加载Driver等操作，再启动kthread进程(pid=2),这些linux系统的内核进程，kthread是所有内核进程的鼻祖；

### 3、HAL
Kernel层加载完毕后，硬件设备驱动与HAL层进行交互。初始化进程管理等操作会启动INIT进程 ，这些在Native层中；

### 4、Init
init进程(pid=1，init进程是所有进程的鼻祖，第一个启动)启动后，会启动adbd，logd等用户守护进程，并且会启动servicemanager(binder服务管家)等重要服务，同时孵化出zygote进程，这里属于C++ Framework，代码为C++程序；

### 5、Zygote
zygote进程是由init进程解析init.rc文件后fork生成，它会加载虚拟机，启动System Server(zygote孵化的第一个进程)；System Server负责启动和管理整个Java Framework，包含ActivityManager，WindowManager，PackageManager，PowerManager等服务；

### 6、App
zygote同时会启动相关的APP进程，它启动的第一个APP进程为Launcher，然后启动Email，SMS等进程，所有的APP进程都有zygote fork生成。


