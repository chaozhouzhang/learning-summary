```java
Base interface for a remotable object, the core part of a lightweight remote procedure call mechanism designed for high performance when performing in-process and cross-process calls.  This interface describes the abstract protocol for interacting with a remotable object.  Do not implement this interface directly, instead extend from Binder.

远程对象的基本接口，是轻量级远程过程调用机制的核心部分，设计用于在执行进程内和跨进程调用时实现高性能。这个接口描述了与远程对象交互的抽象协议。不要直接实现这个接口，而是从Binder扩展。
```
```java
The key IBinder API is transact() matched by Binder.onTransact().  These  methods allow you to send a call to an IBinder object and receive a call coming in to a Binder object, respectively.  This transaction API is synchronous, such that a call to transact() does not return until the target has returned from Binder.onTransact() ; this is the expected behavior when calling an object that exists in the local process, and the underlying inter-process communication (IPC) mechanism ensures that these same semantics apply when going across processes.

关键的IBinder API是由Binder.onTransact()匹配的transact()。这些方法允许您分别发送对IBinder对象的调用和接收对Binder对象的调用。这个事务API是同步的，因此直到目标从Binder.onTransact()返回时，对transact()的调用才会返回;这是调用本地进程中存在的对象时的预期行为，而底层进程间通信(IPC)机制确保在跨进程时应用相同的语义。
```

```java
The data sent through transact() is a Parcel, a generic buffer of data that also maintains some meta-data about its contents.  The meta data is used to manage IBinder object references in the buffer, so that those references can be maintained as the buffer moves across processes.  This mechanism ensures that when an IBinder is written into a Parcel and sent to another process, if that other process sends a reference to that same IBinder back to the original process, then the original process will receive the same IBinder object back.  These semantics allow IBinder/Binder objects to be used as a unique identity (to serve as a token or for other purposes) that can be managed across processes.

通过transact()发送的数据是一个包，它是数据的通用缓冲区，也维护关于其内容的一些元数据。元数据用于管理缓冲区中的IBinder对象引用，以便在缓冲区跨进程移动时维护这些引用。该机制确保当一个IBinder被写入一个包并发送到另一个进程时，如果另一个进程将对同一个IBinder的引用发送回原始进程，那么原始进程将接收回相同的IBinder对象。这些语义允许IBinder/Binder对象被用作唯一的标识(用作令牌或其他用途)，可以跨进程进行管理。
```

```java
	The system maintains a pool of transaction threads in each process that it runs in.  These threads are used to dispatch all IPCs coming in from other processes.  For example, when an IPC is made from process A to process B, the calling thread in A blocks in transact() as it sends the transaction to process B.  The next available pool thread in B receives the incoming transaction, calls Binder.onTransact() on the target object, and replies with the result Parcel.  Upon receiving its result, the thread in process A returns to allow its execution to continue.  In effect, other processes appear to use as additional threads that you did not create executing in your own process.


系统在其运行的每个进程中维护一个事务线程池。这些线程用于调度来自其他进程的所有ipc。例如，当一个IPC是由进程A生成过到达进程B，当它发送事务到进程B，正在调用的线程将在transact()方法处阻塞。因为它把事务处理下一个可用池线程B接收传入的事务,调用Binder.onTransact()在目标对象,并用结果包裹。在收到它的结果后，进程A中的线程返回，允许它继续执行。实际上，其他进程似乎用作在自己的进程中没有执行的其他线程。
```

```
Perform a generic operation with the object.
对对象执行泛型操作。

The action to perform.  This should be a number between FIRST_CALL_TRANSACTION and LAST_CALL_TRANSACTION.
要执行的动作。这应该是FIRST_CALL_TRANSACTION和LAST_CALL_TRANSACTION之间的一个数字。

Marshalled data to send to the target.  Must not be null. If you are not sending any data, you must create an empty Parcel that is given here.
编组数据发送到目标。不能为空。如果您没有发送任何数据，那么您必须创建一个空的包。


Marshalled data to be received from the target.  May be null if you are not interested in the return value.
将从目标接收的数据进行编组。如果您对返回值不感兴趣，则可能为空。

Additional operation flags.  Either 0 for a normal RPC, or FLAG_ONEWAY for a one-way RPC.
额外的操作标志。对于普通RPC是0，对于单向RPC是FLAG_ONEWAY。
```


