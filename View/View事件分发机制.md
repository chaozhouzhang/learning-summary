# View触摸事件分发机制



## 1、触摸事件从硬件触摸屏

1、用户对硬件触摸屏进行操作，会导致硬件产生中断。
2、硬件的驱动程序会处理硬件产生的中断，将数据处理后存储在对应的/dev/input/eventX文件中，完成了事件的数据收集。
3、EventHub，监听、读取/dev/input目录下产生的新事件，并封装成RawEvent结构体供InputReader使用。
4、InputReader : 通过EventHub从/dev/input节点获取事件信息，转换成EventEntry事件加入到InputDispatcher的mInboundQueue队列中。

5、InputDispatcher : 从mInboundQueue队列取出事件，转换成DispatchEntry事件加入到Connection的outboundQueue队列。然后使用InputChannel分发事件到java层。

