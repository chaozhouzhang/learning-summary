# Android触摸事件全过程分析:由产生到Activity.dispatchTouchEvent()


## 触摸事件的产生 : 触摸事件与中断
学习过Linux驱动程序编写的同学可能知道Linux是以中断的方式处理用户的输入事件。触摸事件其实是一种特殊的输入事件。它的处理方式与输入事件相同，只不过触摸事件的提供的信息要稍微复杂一些。
触摸事件产生的大致原理是:用户对硬件进行操作(触摸屏)会导致这个硬件产生对应的中断。该硬件的驱动程序会处理这个中断。不同的硬件驱动程序处理的方式不同，不过最终都是将数据处理后存放进对应的/dev/input/eventX文件中。所以硬件驱动程序完成了触摸事件的数据收集
那/dev/input/eventX中的触摸事件是如何派发到Activity的呢？其实整个过程可以分为两个部分:一个是native(C++)层的处理、一个是java层的处理。我们先来看一下native层是如何处理的。

## 系统对触摸事件的处理
在native层主要是通过下面3个组件来对触摸事件进行处理的，这3个组件都运行在系统服务中:
	•	EventHub : 它的作用是监听、读取/dev/input目录下产生的新事件，并封装成RawEvent结构体供InputReader使用。
	•	InputReader : 通过EventHub从/dev/input节点获取事件信息，转换成EventEntry事件加入到InputDispatcher的mInboundQueue队列中。
	•	InputDispatcher : 从mInboundQueue队列取出事件，转换成DispatchEntry事件加入到Connection的outboundQueue队列。然后使用InputChannel分发事件到java层。
可以用下面这张图描述上面3个组件之间的逻辑:

![](/Users/zhangchaozhou/Study/learning-summary/View/EventHub-InputReader-InputDispatcher.png)


## InputChannel
我们可以简单的把它理解为一个socket, 即可以用来接收数据或者发送数据。一个Window会对应两个InputChannel，这两个InputChannel会相互通信。一个InputChannel会注册到InputDispatcher中, 称为serverChannel(服务端InputChannel)。另一个会保留在应用程序进程的Window中,称为clientChannel(客户端InputChannel)。
下面来简要了解一下这两个InputChannel的创建过程,在Android的UI显示原理之Surface的创建一文中知道,一个应用程序的Window在WindowManagerService中会对应一个WindowState,WMS在创建WindowState时就会创建这两个InputChannel,下面分别看一下他们的创建过程。

## 服务端InputChannel的创建及注册

`WindowManagerService.java`

```
 public int addWindow(Session session...) {
        ...
        WindowState win = new WindowState(this, session, client, token,
            attachedWindow, appOp[0], seq, attrs, viewVisibility, displayContent);
        ...
        final boolean openInputChannels = (outInputChannel != null && (attrs.inputFeatures & INPUT_FEATURE_NO_INPUT_CHANNEL) == 0);
        if  (openInputChannels) {
            win.openInputChannel(outInputChannel);  
        }
    ...
}

void openInputChannel(InputChannel outInputChannel) { //这个 outInputChannel 其实是应用程序获取的inputchannel,它其实就是 inputChannels[1];
    InputChannel[] inputChannels = InputChannel.openInputChannelPair(makeInputChannelName()); //通过native创建了两个InputChannel,实际上是创建了两个socket
    mInputChannel = inputChannels[0]; // 这里将服务端的inputChannel保存在了WindowState中
    mClientChannel = inputChannels[1];
    ....
    mService.mInputManager.registerInputChannel(mInputChannel, mInputWindowHandle); 
}
```


registerInputChannel(..);实际上就是把服务端InputChannel注册到了InputDispatcher中。上图中的InputChannel其实就是在创建一个WindowState时注册的。来看一下InputDispatcher中注册InputChannel都干了什么:

`InputDispatcher.cpp`

```
status_t InputDispatcher::registerInputChannel(const sp<InputChannel>& inputChannel,const sp<InputWindowHandle>& inputWindowHandle, bool monitor) {

    sp<Connection> connection = new Connection(inputChannel, inputWindowHandle, monitor); //利用 inputChannel 创建了一个 connection,简单的理解为socket的链接。

    int fd = inputChannel->getFd();
    mConnectionsByFd.add(fd, connection);

    //把这个 inputChannel 的 fd添加到 Looper中
    mLooper->addFd(fd, 0, ALOOPER_EVENT_INPUT, handleReceiveCallback, this); 

    mLooper->wake();

    return OK;
}
```

即利用InputChannel创建了一个Connection(InputDispatcher会通过这个Connection来向InputChannel发射数据),并且把这个InputChannel添加到mLooper中。
那这里这个mLooper是什么呢？是UI线程的那个Looper吗？这部分我们后面再看，我们先来看一下客户端InputChannel的相关过程。

## 客户端InputChannel的相关逻辑


