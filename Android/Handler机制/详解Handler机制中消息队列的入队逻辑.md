## 1、源码分析
具体分析请见代码注释：
```java
/**
 * 消息队列是以执行时间为序的优先级队列
 *
 * @param msg
 * @param when
 * @return
 */
boolean enqueueMessage(Message msg, long when) {

    //入队消息没有绑定Handler
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    //入队消息已经在使用中
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    //获取同步锁
    synchronized (this) {
        //入队消息正在被取消发送
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            //回收入队消息
            msg.recycle();
            return false;
        }
        //标记入队消息为正在使用中
        msg.markInUse();
        //入队消息的执行时间
        msg.when = when;
        //获取消息队列的队首消息
        Message p = mMessages;
        //是否需要唤醒线程
        boolean needWake;
        //如果队列首部为null，也就是队列为空
        //或如果入队消息的执行时间为0，也就是入队消息需要马上执行
        //或如果入队消息的执行时间小于，也就是早于队首消息的执行时间
        if (p == null || when == 0 || when < p.when) {
            // 新头，唤醒事件队列如果阻塞。
            // New head, wake up the event queue if blocked.
            // 入队消息的指针指向队首消息
            msg.next = p;
            // 入队消息成为新的队首消息
            mMessages = msg;
            // 且如果线程已经阻塞
            // 则需要唤醒
            needWake = mBlocked;
        } else {
            // 插入到队列中间。通常我们不需要唤醒事件队列，除非在队列的顶部有一个屏障，并且消息是队列中最早的异步消息。
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.

            // 如果线程已经阻塞
            // 且如果队列的队首消息的Handler为空，也就是队首消息是同步屏障消息
            // 且如果入队消息是异步的，也就是可以通过同步屏障
            // 则需要唤醒
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            //前一条消息
            Message prev;
            // 循环
            for (; ; ) {
                //获取队列首条消息
                prev = p;
                //获取下一条消息
                p = p.next;
                //如果下一条消息是空的，或者消息的执行时间早于下一条消息的执行时间，则退出循环
                if (p == null || when < p.when) {
                    break;
                }
                // 但如果线程已经阻塞
                // 且如果队列中有异步消息，可以穿过同步屏障
                // 则不需要唤醒

                // 如果线程已经阻塞
                // 且如果线程中没有异步消息，没办法穿过同步屏障
                // 就需要唤醒
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            // 将入队消息指向获取到的消息
            msg.next = p; // invariant: p == prev.next
            // 将前一条消息的指针指向入队消息
            prev.next = msg;
        }

        // 我们可以假设mPtr != 0，因为m是假的。
        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            //如果需要唤醒，则进行唤醒
            nativeWake(mPtr);
        }
    }
    return true;
}
```
## 2、消息是分哪些情况入队的？如何入队？
我们剖除入队规则、同步锁、同步屏障消息、异步消息、唤醒规则等逻辑，将入队的逻辑代码抽出，得到：
```java
public class Message {
    public Object obj;
    public long when;
    public Message next;
}
```
```java
public class MessageQueue {
    public Message mMessages;

    public void enqueueMessage(Message msg, long when) {
        msg.when = when;
        Message p = mMessages;
        if (p == null || when == 0 || when < p.when) {
            msg.next = p;
            mMessages = msg;
        } else {
            Message prev;
            for (; ; ) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
            }
            msg.next = p;
            prev.next = msg;
        }
    }
}
```
### 2.1、往空队列插入消息
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Handler%E6%9C%BA%E5%88%B6/Handler-%E5%BE%80%E7%A9%BA%E9%98%9F%E5%88%97%E6%8F%92%E5%85%A5%E6%B6%88%E6%81%AF.png?raw=true)
### 2.2、在队列头插入消息
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Handler%E6%9C%BA%E5%88%B6/Handler-%E5%9C%A8%E9%98%9F%E5%88%97%E5%A4%B4%E6%8F%92%E5%85%A5%E6%B6%88%E6%81%AF.png?raw=true)
### 2.3、在队列尾插入消息
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Handler%E6%9C%BA%E5%88%B6/Handler-%E5%9C%A8%E9%98%9F%E5%88%97%E5%B0%BE%E6%8F%92%E5%85%A5%E6%B6%88%E6%81%AF.png?raw=true)
### 2.4、在队列中插入消息
![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Handler%E6%9C%BA%E5%88%B6/Handler-%E5%9C%A8%E9%98%9F%E5%88%97%E4%B8%AD%E6%8F%92%E5%85%A5%E6%B6%88%E6%81%AF.png?raw=true)



## 3、消息入队时，什么情况下需要主动唤醒线程？

### 3.1、队列中没有任何消息，且线程阻塞
此时新消息入队后便主动唤醒线程，无论新消息是同步消息、异步消息。 

### 3.2、队首的消息执行时间未到，且线程阻塞
如果在阻塞时长未耗尽时，就新加入早于队首消息处理时间的消息，需要主动唤醒线程。
1、如果入队消息的执行时间为0，也就是入队消息需要马上执行。
2、如果入队消息的执行时间小于队首消息的执行时间，也就是入队消息要早于队首消息执行。

### 3.3、队首消息是同步屏障消息，并且队列中不含有异步消息，且线程阻塞
如果新加入的消息仍然是晚于队首同步障碍器处理时间，那么这次新消息的发布在next()层面上是毫无意义的，我们也不需要唤醒线程。
只有在新加入早于队首同步障碍器处理时间的同步消息时，或者，新加入异步消息时（不论处理时间），才会主动唤醒被next()阻塞的线程。 

### 3.4、队首消息是同步屏障消息，队列中含有异步消息但执行时间未到，切线程阻塞
因为队首同步障碍器的缘故，无论新加入什么同步消息都不会主动唤醒线程。
即使加入的是异步消息也需要其处理时间早于设定好唤醒时执行的异步消息，才会主动唤醒。

欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！
![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)

