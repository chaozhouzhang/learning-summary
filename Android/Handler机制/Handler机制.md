

如何通过Handler进行子线程间的通信？

## Handler机制

首先来看android源码的主线程是如何与其他线程进行通信的：

```java
public static void main(String[] args) {
    //准备好主线程的消息循环器
    Looper.prepareMainLooper();
    
    //主线程，绑定到Application中去
    ActivityThread thread = new ActivityThread();
    thread.attach(false);

    //获取主线程的消息处理器mH
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    //循环取出消息
    Looper.loop();
}
```
### 1、Looper
先获取当前主线程的Looper：
```
Looper.prepareMainLooper();
```
创建后的Looper会使用ThreadLocal存储起来，每个线程都有对应的唯一的Looper：
```
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```
创建Looper的同时，会绑定当前线程，并且会创建消息队列，用来存储其他线程发送过来的消息：
```
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```
获取当前线程的Looper：
```
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```
### 2、Handler
获取Looper后，就可以根据Looper创建对应的Handler，表示使用当前Handler来处理Looper取出来的Message：
```
public Handler(@NonNull Looper looper) {
    this(looper, null, false);
}
```
创建Handler后就需要使用Looper去循环取出MessageQueue的Message：
```
Looper.loop();
```
一旦获取到Message，Message对应的Handler就会把Message分发出去：
```
msg.target.dispatchMessage(msg);
```
### 3、Message
Handler发送Message：
```
public final boolean sendMessage(@NonNull Message msg) {
    return sendMessageDelayed(msg, 0);
}
```
Message绑定Handler：
```
private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
        long uptimeMillis) {
    msg.target = this;
    msg.workSourceUid = ThreadLocalWorkSource.getUid();

    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```
将Message插入消息队列MessageQueue，等待被Looper取出然后被Handler分发出去：
```
queue.enqueueMessage(msg, uptimeMillis)
```
## 流程图
下图是笔者根据源码画出的Handler消息机制的流程图：

![Handler机制](https://raw.githubusercontent.com/chaozhouzhang/learning-summary/master/Android/Handler%E6%9C%BA%E5%88%B6.png)
## 子线程间的消息通信
最后我们依样画葫芦，进行子线程间的通信：
```java
public class HandlerActivity extends AppCompatActivity {

    private static Handler sHandler;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Message message = new Message();
                message.obj = "Message From A.";
                //发送消息
                sHandler.sendMessage(message);
            }
        }, "A").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                //创建Looper和MessageQueue
                Looper.prepare();
                //创建Handler
                sHandler = new Handler(Looper.myLooper()) {
                    @Override
                    public void handleMessage(@NonNull Message msg) {
                        super.handleMessage(msg);
                        //处理消息
                        System.out.println("Message To B:" + msg.obj);
                    }
                };
                //循环Looper
                Looper.loop();
            }
        }, "B").start();
    }
}
```

欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！

![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)

