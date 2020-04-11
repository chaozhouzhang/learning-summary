MessageQueue
  MessageQueue（消息队列）是Message（消息）的管理者，它负责保存消息的集合，执行消息入队、出队等操作，同时提供SyncBarrier（同步障碍器）与IdleHandler（闲时任务）机制。SyncBarrier机制允许我们暂停部分Message的出队，而IdleHandler机制允许我们在没有消息需要出队处理时执行一些简单的任务。
1.MessageQueue的创建
消息队列只含一个构造方法，其代码如下：
MessageQueue(boolean quitAllowed) {
      mQuitAllowed = quitAllowed;
      mPtr = nativeInit();
}
 
  在Java中如果定义类的方法时不设置方法的访问权限，那么方法默认的访问权限不是public，不是private，也不是protected。方法默认的访问权限允许同一个包下的类访问该方法，但不允许子类继承与访问。翻看调用记录可发现，消息队列所在的Android.os包下只有Looper类调用过消息队列的构造方法，而消息队列中并没有诸如缓存池之类的结构可以获取已经构造好的消息队列。我们可以断定消息队列是依附在Looper上的消息对象集合，想要获得消息队列必须通过Looper（Looper.myQueue()）。    quitAllowed指定消息队列是否允许退出，其值由Looper类指定，在Looper关联的线程是主线程时才为false
 
2.同步消息与异步消息
  同异步消息的区别有两点。第一点，同步消息自始至终都会按照顺序执行（when相同的消息，哪个先入队就先执行哪个），异步消息的执行顺序则完****全不确定。第二点，同步消息会被同步障碍器拦截而异步消息不会受到影响。    第一点出自官方文档（Message.isAsynchronous()注释），暂未找到可以佐证的代码块。第二点，我们可以从SyncBarrier的工作原理中得到佐证。    通常我们会把中断消息、事件消息等较为重要的Message设置为异步消息，以保证系统能够尽早处理这些消息。
 
3.SyncBarrier（同步障碍器）
先上结论：SyncBarrier是特殊的消息对象，其特征是target字段为null且arg1字段保存token，其作用是阻碍消息队列使其在处理普通消息时直接跳过位于SyncBarrier后的所有 同步 消息。  我们先来看看SyncBarrier是怎么定义的：

 /**
     * 将同步障碍器加入消息队列。如果此时消息队列处于阻塞状态也不需要唤醒，因为障碍器本身的目的就是
     * 阻碍消息队列的循环处理。（可以假设一下为什么阻塞，各种阻塞场景下需不需要唤醒）
     * @param when 同步障碍器从何时起效（这个时间是自系统启动开始算起，到指定时间的不包含深度睡
     *             眠的毫秒数）。
     * @return  新增的同步障碍器token，用于{@link #removeSyncBarrier(int) }移除障碍器时使用
     * */
    int postSyncBarrier(long when) {
        synchronized (this) {
            final int token = mNextBarrierToken++;
            //从消息池取出一个消息，并将其设置为同步障碍器（target为null，且arg1保存token的消息）
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;

            //找到msg在消息队列中的位置（消息队列按照when从小到大排列），并把msg插入其中
            …………（省略）
            return token;
        }
    }

 
 
  看到这里想必大家有一个疑问：难道普通消息的terget不可能是null吗？我们目光移到MessageQueue.enqueueMessage(Message，long)方法上，这个方法是唯一能够从队列外部将一个普通消息加入队列的方法，方法体中的第一行便有这样一段代码：
if (msg.target == null) {
      throw new IllegalArgumentException("Message must have a target.");
}
 
  如果准备加入队列的Message.target为null，加入操作会抛出异常。因此我们可以断定，消息队列中target为null的消息一定是SyncBarrier。
 
 
继续搜索：

 /**
     *得到下一个等待处理的消息。如果当前消息队列为空或者下一个消息延时时间未到则阻塞线程。
     *
     * @return  <em>null</em> 消息队列已经退出或者被废弃
     */
Message next() {
   …………（略）
   for (;;) {
      …………（略）
      synchronized (this) {
      // Try to retrieve the next message.  Return if found.
      //now等于自系统启动以来到此时此刻，非深度睡眠的时间
      final long now = SystemClock.uptimeMillis();
      Message prevMsg = null;
      Message msg = mMessages;

      //如果当前队首的消息时设置的同步障碍器
      if (msg != null && msg.target == null) {
           // 因为同步障碍器的原因而进入该分支。分支找到下一个异步消息之后才会结束。
           do {
                prevMsg = msg;
                msg = msg.next;
           } while (msg != null && !msg.isAsynchronous());
      }
      …………（略）
  }//for(;;)结束
}

 
 
  如果队首是SyncBarrier，next()在寻找下一个待处理的普通消息时遇到同步消息会直接跳过，遇到异步消息才会继续执行。这段代码证实了我们之前的结论。继续搜索"target == null"，除了入队抛异常时又出现了一次外再也没出现过了。
 
异步消息和SyncBarrier
为了让View能够有快速的布局和绘制，Android中定义了一个SyncBarrier的概念，当View在绘制和布局时会向MessageQueue中添加了Barrier(监控器)，这样后续的消息队列中的同步的消息将不会被执行，以免会影响到UI绘制，但是只有异步消息才能被执行。
    所谓的异步消息也只是体现在这，添加了Barrier后，消息还可以继续被执行，不会被推迟运行。
如何使用异步消息？
异步消息是不能使用的，因为相关设置都是hide。
handler的相关构造函数，hide
Mesasge.setAsynchronous(boolean)，hide
 
假设我们可以使用，则 在创建Handler时，可以在构造方法上传递参数true，如下：
    public Handler(boolean async) {
        this(null, async);
}
 
 
在Handler上设置异步的话，那么使用此Handler发送的消息都将被设置Mesasge#setAsynchronous(true)

    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {  
        msg.target = this;  
        if (mAsynchronous) {  
            msg.setAsynchronous(true);  
        }  
        return queue.enqueueMessage(msg, uptimeMillis);  
}  

 
 
当ViewRootImpl开始measure和layout ViewTree时，会向主线程的Handler添加Barrier

    void scheduleTraversals() {  
        if (!mTraversalScheduled) {  
            mTraversalScheduled = true;  
            mTraversalBarrier = mHandler.getLooper().postSyncBarrier();  
            mChoreographer.postCallback(  
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);  
            scheduleConsumeBatchedInput();  
        }  
    }  

    void unscheduleTraversals() {  
    if (mTraversalScheduled) {  
        mTraversalScheduled = false;  
        mHandler.getLooper().removeSyncBarrier(mTraversalBarrier);  
        mChoreographer.removeCallbacks(  
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);  
} 

 
 
scheduleTraversals()方法都会在requestLayout时会被调用，所有导致布局变化的都会触发。例如：ViewGroup#addView,removeView或者View 从VISIBLE-->GONE,或者GONE-->VISIBLE等。
 
4.next()
  next()是消息队列中最为核心的方法，Looper从消息队列中取消息就是通过next()实现的。next()保证每次调用都能让Looper得到一个消息，除非消息队列正在退出或者已经废弃（此时返回null）。也就是说如果暂时取不出消息，next()并不会返回！此时为了节省资源，next()会根据消息队列的情况设定阻塞时长然后再阻塞线程。   消息队列（未退出、未被废弃）在以下四种情况下，next()会选择阻塞线程：    1）队列中没有任何消息 – 永久阻塞：这个时候不能返回null，因为next()的目的是取出一个消息，队列中现在没有消息并不代表一段时间后也没有消息。消息队列还在可用中，随时都有可能有Handler发布新的消息给它。那么问题来了，为了节省资源准备阻塞线程但是多少时间后唤醒它呢？臆断一个时长并不是很好的解决方案。我们知道消息队列是用来管理消息的，既然确定不了阻塞时长那么不如先永久阻塞，等新消息入队后主动唤醒线程。    2）队首的消息执行时间未到 – 定时唤醒：每个消息的when字段都给出了希望系统处理该消息的时刻。如果在next()方法取消息时，发现消息队列的队首消息处理时间未到，next()同样需要阻塞。因为消息队列是按照消息的when字段从小到大排列的，如果队首消息的处理时间都没到那么整个队列中都没有能够立即取出的消息。这个时候因为知道下一次处理的具体时间，结合当前时间就可以确定阻塞时长。    3）队首消息是同步障碍器（SyncBarrier），并且队列中不含有异步消息 – 永久阻塞：因为对消息队列施加了同步障碍，所有晚于队首同步障碍器处理时间的同步消息都变得不可用，next()在选取返回消息时会完全忽略这些消息。这和第一种情况相似，所以采取的阻塞方案也是永久阻塞。    4）队首消息是同步障碍器（SyncBarrier），队列中含有异步消息但执行时间未到 – 定时唤醒：因为对消息队列施加了同步障碍，所有晚于队首同步障碍器处理时间的同步消息都变得不可用，next()在选取返回消息时会完全忽略这些消息。也就是说对于next()，它只会考虑队列中的异步消息。这和第二种情况相似，所以采取的阻塞方案是设置阻塞时长再阻塞。
 
在这个基础上，我们再以验证的方式来看next()方法的源代码：

 
5.IdleHandler（闲时任务）
  IdleHandler允许我们在消息队列空闲时执行一些不耗时的简单任务。

我们来看看消息队列是怎么使用IdleHandler的，首先目光转到成员变量上，我们可以发现有这样两个成员：

  mIdleHandlers可以理解成是一个IdleHandler的总列表，每次next()将要执行IdleHandler时都会从这个总列表中取出所有的IdleHandler。mPendingIdleHandlers指定哪些IdleHandler需要在本次执行中完成，每次next()将要执行IdleHandler时都会从mIdleHandlers拷贝IdleHandler的总列表到mPendingIdleHandlers中。
 
下面这一段代码是MessageQueue.next()中用于处理IdleHandler的代码段：

  执行IdleHandler其实就是调用其queueIdle()方法，queueIdle()如果返回false，next()方法会将该IdleHandler从mIdleHandlers中删除。这样的话，下一次next()方法再执行IdleHandler时就不会再重复执行它了。    需要特别提醒的是，虽然next()方法是一个无限for循环，但是每次调用next()都只会执行一次mIdleHandlers中的闲时任务。因为在上面的代码段之前有这样一段：

 
 
6.主动唤醒
  针对上一节提到的四种阻塞情况，我们来分析一下对应情况下什么时候才需要主动唤醒：    1）队列中没有任何消息 – 永久阻塞：新消息入队后便主动唤醒线程，无论新消息是同步消息、异步消息。    2）队首的消息执行时间未到 – 定时唤醒：如果在阻塞时长未耗尽时，就新加入早于队首消息处理时间的消息，需要主动唤醒线程。    3）队首消息是同步障碍器（SyncBarrier），并且队列中不含有异步消息 – 永久阻塞：如果新加入的消息仍然是晚于队首同步障碍器处理时间，那么这次新消息的发布在next()层面上是毫无意义的，我们也不需要唤醒线程。只有在新加入早于队首同步障碍器处理时间的同步消息时，或者，新加入异步消息时（不论处理时间），才会主动唤醒被next()阻塞的线程。    4）队首消息是同步障碍器（SyncBarrier），队列中含有异步消息但执行时间未到 – 定时唤醒：因为队首同步障碍器的缘故，无论新加入什么同步消息都不会主动唤醒线程。即使加入的是异步消息也需要其处理时间早于设定好唤醒时执行的异步消息，才会主动唤醒。
 
  上面的讨论中，我们只考虑了新增普通消息时，是否需要主动唤醒阻塞中的线程。现在我们来考虑一下，移除普通消息时是否应该唤醒。第一种情况没有消息跳过。第二种情况和第四种情况下，假设我们移除设定好下次被动唤醒时执行的消息，线程被唤醒后就会因为没有需要处理的消息而再次进入阻塞，并不会错过消息所以不需要主动唤醒。第三种情况下移除设定好下次被动唤醒时执行的消息，线程虽然会再次进入阻塞但并不会错过消息，也不需要主动唤醒。所以，移除普通消息在任何情况下都不需要主动唤醒线程。    如果是新增和移除同步障碍器呢？无论哪种情况，新增的同步障碍器都会在被动唤醒时发挥同步障碍的作用，不会因为没有主动唤醒而多处理不该处理的消息，所以新增同步障碍器之后不需要主动唤醒线程。针对第三种和第四种情况，移除队首障碍器能够使本不可取出的同步消息变得可用，需要主动唤醒线程重新判断是否能够取出消息或者是否需要缩短阻塞时长。除非新的队首消息还是同步障碍器才不需要唤醒！    至于加长阻塞时长使线程不会被无谓地被动唤醒（因为移除消息或者障碍器），这个设定至少在API22之前还没写入到代码中。
 
就着上面给出的结论，我们来看看Android是怎么实现这些设定的。首先目光回到MessagqeQueue.enqueueMessage()，来看看新增普通消息时是怎么判断是否需要主动唤醒的：

  在这个方法中，出现了一种我们前文都没提到的一种主动唤醒情况 —— when==0（立即执行），实际上这也是毫无疑问需要主动唤醒的一种情况。对于第一种情况，新加入消息肯定需要主动唤醒；对于第二种，不主动唤醒会错过；对于第三、第四种情况，队首的同步障碍器不能影响早于它执行的消息，所以新加入when为0的消息无论如何都能够执行，如果不主动唤醒也会错过！所以，无论什么情况，只要新入队的消息when字段为0，都要主动唤醒线程！    移除消息的MessageQueue.removeMessages()系列和MessageQueue.removeCallbacksAndMessages()方法，虽然可能导致线程下次被动唤醒时没有消息执行，但是都不会错过消息所以不需要主动唤醒。    接下来目光转到同步障碍器上，MessageQueue.removeSyncBarrier()代码：

  "needWake = mMessages == null || mMessages.target != null"，这个语句含有一种有趣的情况。当消息队列中不含普通消息只含一个同步障碍器时，移除这个障碍器后整个消息队列都空了。按理说，移除之前next()线程已经处于无限阻塞中，移除后再唤醒结果还是无线阻塞。从消息处理上来讲，这是一个可以轻松避免且毫无意义的唤醒。从空闲任务的管理上来讲，next()方法在阻塞线程之前都会执行空闲任务然后再迭代一次判断是否阻塞，阻塞后再唤醒也不可能在本次next()中再执行一次空闲任务，依然是一个可以轻松避免且毫无意义的唤醒。    另外一点是，这段代码并没有融合mBlocked（记录当前线程是否阻塞）变量的值。可能出现线程未阻塞时主动唤醒线程的无谓举动。    这两个问题也许在看到本地方法nativeWake()的定义之后才能解开，暂时留下疑问。
 
7.消息队列的退出与废弃
  当Looper对象退出循环处理时，会调用MessageQueue的同包成员方法quit(safe)通知消息队列开始退出操作。如果boolean型的参数safe是true，消息队列会清除when晚于当前时间的所有同步/异步消息与同步障碍器，留下本应处理完的消息继续处理；如果safe是false，则完全不顾虑，清除消息队列中的所有消息。    在next()方法执行过程中，如果处理完队列中全部消息后发现该消息队列的quit()方法被调用过，则直接调用dispose()废弃消息队列并返回null给Looper。当GC回收消息队列之前，会调用消息队列重载的finalize()方法，在这个方法中同样能够执行废弃消息队列的操作（如果还未废弃）。


