# Android Launcher启动流程概览

在上一篇文章《Android操作系统启动流程概览》中，我们了解到当电源键按下后：

|Android操作系统启动流程|
|----|
|1、BootROM代码从ROM中预先定义的位置开始执行，引导BootLoader加载到RAM中执行。|
|2、BootLoader开始执行，检测外部RAM并帮助加载程序后，执行跳转到Linux内核。|
|3、Linux内核启动，初始化中断控制器、设置内存保护、缓存和调度等，并启动init进程。|
|4、init进程启动，挂载目录、运行init.rc脚本，并启动zygote进程、service manager进程、其他守护进程等。|
|5、zygote进程启动，执行并初始化Dalvik VM/ART，同时启动系统服务System Server。|
|6、System Server系统服务启动，将启动所有的android服务，并发送ACTION_BOOT_COMPLETED广播。|


