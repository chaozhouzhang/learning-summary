### 源码解析
源码解析请直接见代码注释：
```java
@UnsupportedAppUsage
Message next() {
    // 如果消息循环已经退出并已被释放，则返回此处。
    // Return here if the message loop has already quit and been disposed.
    // 如果应用程序在退出后试图重新启动looper(不支持该操作)，就会发生这种情况。
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        //当调用了dispose就会出现 mPtr == 0
        return null;
    }

    // 正在挂起的空闲句柄数量：-1只在第一次迭代期间
    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    // 下一次轮询超时毫秒数
    int nextPollTimeoutMillis = 0;
    //循环
    for (; ; ) {
        // 下一次轮询超时毫秒数!=0
        if (nextPollTimeoutMillis != 0) {
            // 将当前线程中挂起的任何绑定器命令刷新到内核驱动程序。
            // 在执行可能会阻塞很长时间的操作之前调用此方法非常有用，可以确保释放了所有挂起的对象引用，以防止进程持有对象的时间超过需要的时间。
            Binder.flushPendingCommands();
        }
        //native层阻塞CPU
        nativePollOnce(ptr, nextPollTimeoutMillis);
        //获取同步锁
        synchronized (this) {
            // 尝试检索下一个消息，如果找到就返回。
            // Try to retrieve the next message.  Return if found.
            // 返回启动后的毫秒数，不计算深度睡眠的时间。
            final long now = SystemClock.uptimeMillis();
            // 上一个消息
            Message prevMsg = null;
            // 对首消息
            Message msg = mMessages;
            // 队首为同步屏障消息
            if (msg != null && msg.target == null) {
                // 被同步屏障消息阻挡，寻找队列中的下一个异步消息。
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                    //如果所循环到的消息为异步消息，则退出循环
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                // 如果当前时间小于消息的执行时间，也就是消息还没准备好发送
                if (now < msg.when) {
                    // 下一个消息还没准备好。设置一个超时时间，当消息准备好了，就来唤醒它。并且该时间不能超过整型的最大值。
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 获取到了一条消息
                    // Got a message.
                    mBlocked = false;
                    // 将该条消息出队列
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    //标记该条消息正在使用中
                    msg.markInUse();
                    //返回该条消息
                    return msg;
                }
            } else {
                // 队列中没有消息了，标记下一次轮询超时毫秒数为-1
                // No more messages.
                nextPollTimeoutMillis = -1;
            }

            // 现在所有挂起的消息都已处理完毕，请处理quit消息。
            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }

            // 如果第一次空闲，则获得要运行的空闲组的数量。
            // If first time idle, then get the number of idlers to run.
            // Idle句柄仅在队列为空或队列中的第一个消息(可能是一个障碍)将在将来处理时运行。
            // Idle handles only run if the queue is empty or if the first message
            // in the queue (possibly a barrier) is due to be handled in the future.
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                // 如果挂起的空闲句柄数小于0，也就是-1；并且当前的队列为空或者当前的非睡眠的启动毫秒数小于队列的第一个消息的目标分发时间；则获取挂起的空闲句柄数。
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // 没有需要运行的空闲句柄，循环并且等待更多。
                // No idle handlers to run.  Loop and wait some more.
                // 阻塞
                mBlocked = true;
                // 继续循环操作
                continue;
            }

            if (mPendingIdleHandlers == null) {
                // 如果挂起的空闲句柄为空，则取挂起的空闲句柄数和4的最大值来创建
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }

            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        // 运行空闲的句柄
        // Run the idle handlers.
        // 我们只有在第一个迭代期间曾经到达过这个代码块
        // We only ever reach this code block during the first iteration.
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            // 释放handler的引用
            mPendingIdleHandlers[i] = null; // release the reference to the handler

            boolean keep = false;
            try {
                //判断是否保持空闲处理程序处于活动状态
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }

            if (!keep) {
                //如果不保持空闲处理程序处于活动状态，则移除
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // 将空闲处理程序计数重置为0，这样我们就不会再次运行它们。
        // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;

        // 在调用空闲处理程序时，可能已经传递了新消息，因此请返回并再次查找未等待的消息。
        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}
```
## 消息是分哪些情况出队的？如何出队？

我们剖除出队规则、同步锁、唤醒规则、取消发送、IdleHandler等逻辑，将出队的逻辑代码抽出，得到：
```
public class Handler {

}
```
```java
public class Message {
    public Object obj;
    public long when;
    public Message next;
    public boolean isAsynchronous;

    public boolean isAsynchronous() {
        return isAsynchronous;
    }

    public Handler target;
}
```
```java
public class MessageQueue {
    public Message mMessages;

    public Message next() {
        final long now = System.currentTimeMillis();
        Message prevMsg = null;
        Message msg = mMessages;
        if (msg != null && msg.target == null) {
            do {
                prevMsg = msg;
                msg = msg.next;
            } while (msg != null && !msg.isAsynchronous());
        }
        if (msg != null) {
            if (now < msg.when) {
                //尚未到消息执行时间
            } else {
                if (prevMsg != null) {
                    //队首是同步屏障消息，此处取出同步屏障消息后的异步消息
                    prevMsg.next = msg.next;
                } else {
                    //取出队首的同步消息
                    mMessages = msg.next;
                }
                msg.next = null;
                return msg;
            }
        } else {
            //空队列
        }
        return null;
    }
}
```

### 1、取出队首的同步消息

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Handler%E6%9C%BA%E5%88%B6/Handler-%E5%8F%96%E5%87%BA%E9%98%9F%E9%A6%96%E7%9A%84%E5%90%8C%E6%AD%A5%E6%B6%88%E6%81%AF.png?raw=true)

### 2、队首是同步屏障消息，此处取出同步屏障消息后的异步消息

![](https://github.com/chaozhouzhang/learning-summary/blob/master/Android/Handler%E6%9C%BA%E5%88%B6/Handler-%E5%8F%96%E5%87%BA%E5%B1%8F%E9%9A%9C%E5%90%8E%E7%9A%84%E5%BC%82%E6%AD%A5%E6%B6%88%E6%81%AF.png?raw=true)

## IdleHandler是什么？它有什么作用呢？

```
/**
 * 回调接口，用于发现线程将在何时阻塞以等待更多消息。
 * Callback interface for discovering when a thread is going to block
 * waiting for more messages.
 */
public static interface IdleHandler {
    /**
     * 当消息队列耗尽消息并将等待更多消息时调用。返回true以保持空闲处理程序处于活动状态，返回false则删除它。如果队列中仍然有未处理的消息，可以调用此方法，但是它们都被安排在当前时间之后进行分发。
     * <p>
     * Called when the message queue has run out of messages and will now
     * wait for more.  Return true to keep your idle handler active, false
     * to have it removed.  This may be called if there are still messages
     * pending in the queue, but they are all scheduled to be dispatched
     * after the current time.
     */
    boolean queueIdle();
}
```

IdleHandler是告知线程已经是处于阻塞状态空闲的接口，我们可以实现这个接口，并且实现方法返回TRUE的时候表示消息线程一旦空闲就会执行实现的操作，返回false的时候表示无论线程何时空闲，实现的操作只会执行一次。



使用Idle可以优化Activity的启动时间，把在onResume以及其之前的调用的但非必须的事件（如某些界面View的绘制）挪出来放在实现IdleHandler接口的方法中（即绘制完成以后）去调用。


```
@Override
protected void onResume() {
    super.onResume();
    Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
        @Override
        public boolean queueIdle() {
            //执行操作
            return false;
        }
    });
}
```

欢迎关注Android技术堆栈，专注于Android技术学习的公众号，致力于提高Android开发者们的专业技能！
![Android技术堆栈](https://mmbiz.qpic.cn/mmbiz_jpg/MADc6NnIysDjTRbKsg6y2G5eqqQkPDiak4V8jqKLmntDgAfFE8LOibxnSdfJESLJEM8ibrN9RGiamib4rYCt3cU08aQ/0?wx_fmt=jpeg)

