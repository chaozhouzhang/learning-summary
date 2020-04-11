```
boolean enqueueMessage(Message msg, long when) {
    //Message必须要绑定Handler
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    //Message必须还未使用
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

	//获取同步锁
    synchronized (this) {
        if (mQuitting) {
        //正在取消发送
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            //回收消息
            msg.recycle();
            return false;
        }
			//标记正在使用
        msg.markInUse();
        //消息的分发时间
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```


