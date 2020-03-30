# 对象的内存布局



在HotSpot虚拟机中，对象在内存中存储的布局分为对象头Header、实例数据Instance Data、对齐填充Padding。

|Java对象|大小|
|----|----|
|对象头 Header|8字节的1倍或2倍|
|实例数据 Instance Data|与对齐填充的总大小为8字节的整数倍|
|对齐填充 Padding|与实例数据的总大小为8字节的整数倍|

## 1、对象头

|对象头|大小|
|----|----|
|Mark Word|4字节(32位虚拟机)|
|Class对象指针|4字节|
|如果对象是数组，则包含数组长度 Length|4字节|

### 1.1、Mark Word
在HotSpot虚拟机中，对象头的第一部分用于存储对象自身的运行时数据，例如哈希码、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，也称Mark Word。

HotSpot虚拟机32位虚拟机Mark Word存储内容：

|Mark Word存储内容|标志位|状态|
|----|----|----|
|对象哈希码、对象的分代年龄|01|未锁定|
|指向栈中锁记录的指针|00|轻量级锁定|
|指向互斥量/重量级锁的指针|10|膨胀/重量级锁定|
|空，不需要记录信息|11|GC标志|
|偏向线程ID、偏向时间戳Epoch、对象分代年龄|01|可偏向锁|

HotSpot虚拟机32位虚拟机Mark Word存储内容示例图：

![Mark Word](https://mmbiz.qpic.cn/mmbiz_png/MADc6NnIysCeAZQq27fpzr1XEJ6xSvAU1yjlv8LutIlCdz9XjMUOyahPhsjfJS4y4WRaDic7eXe977jWIuTRKVQ/0?wx_fmt=png)
### 1.2、Class对象指针
在HotSpot虚拟机中，对象头的第二部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

### 1.3、数组长度Length
在HotSpot虚拟机中，如果对象是数组，则包含数组长度，作为对象头的第三部分。

## 2、实例数据
实例数据部分是对象真正存储的有效信息，也是程序代码中所定义的各种类型的字段内容。

## 3、对齐填充
对齐填充并不是必然存在的，无特别含义，起着占位符的作用，HotSpotVM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，也就是对象的大小必须是8字节的整数倍，对象头正好是8字节的倍数，所以当对象实例数据部分没有对齐，就需要通过对齐填充来补全。

欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！

![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)

