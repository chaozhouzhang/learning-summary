第五章 Invocation API
5.1 简介
Invocation API允许软件提供商在原生程序中内嵌Java虚拟机。因此可以不需要链接任何Java虚拟机代码来提供Java-enabled的应用程序。
以下代码演示如何使用：
    #include <jni.h>       /* where everything is defined */
    //...
    JavaVM *jvm;       /* denotes a Java VM */
    JNIEnv *env;       /* pointer to native method interface */
    JavaVMInitArgs vm_args; /* JDK/JRE 6 VM initialization arguments */
    JavaVMOption* options = new JavaVMOption[1];
    options[0].optionString = "-Djava.class.path=/usr/lib/java";

    vm_args.version = JNI_VERSION_1_6;
    vm_args.nOptions = 1;
    vm_args.options = options;
    vm_args.ignoreUnrecognized = false;

    /* load and initialize a Java VM, return a JNI interface
     * pointer in env */
    JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args);
    delete options;

    /* invoke the Main.test method using the JNI */
    jclass cls = env->FindClass("Main");
    jmethodID mid = env->GetStaticMethodID(cls, "test", "(I)V");
    env->CallStaticVoidMethod(cls, mid, 100);

    /* We are done. */
    jvm->DestroyJavaVM();

创建虚拟机
JNI_CreateJavaVM() 函数载入和初始化一个Java虚拟机。调用该函数的线程被视为是主线程（main thread）。

attach到虚拟机
JNI接口指针(JNIEnv)只在当前线程有效，如果需要在另一个线程访问Java虚拟机，必须先调用 AttachCurrentThread() 来将自己 attach 到虚拟机来获得JNI接口指针（JNIEnv）
被acttach到的线程必须有足够的stack空间来执行一定的工作。每个线程分配多少栈空间根据系统而不同。

detach虚拟机
一个attach到虚拟机的本地线程必须在退出前调用 DetachCurrentThread() 来和虚拟机detach。如果还有Java方法在call stack中，则这个线程不能detach。

unload虚拟机
使用 JNI_DestroyJavaVM() 函数来卸载（unload）一个Java虚拟机
虚拟机会等待（阻塞），直到当前线程成为唯一的非守护进程的用户进程(the only non-daemon user thread)，才真正执行 unload 操作。
用户进程(user thread)包括:
	•	Java线程(java threads)
	•	attached到虚拟机的本地线程(attached native threads)
为什么要做这样的限制（强制等待），是因为Java线程和native线程可能会hold住系统资源，例如锁，窗口等资源，而虚拟机不能自动释放这些资源。通过限制当前线程是唯一的运行中的用户线程才unload虚拟机，则将释放这种系统资源的任务交给程序员自己来负责了。

5.2 库和版本管理
在 JDK/JRE 1.1，一旦本地库被加载，它对所有classLoader都可见。因此两个被不同的classLoader加载的类可能链接到同一个本地方法。这会出现两个问题：
	•	一个类可能错误的链接到了另一个classLoader载入的同名本地库。
	•	本地方法可以轻松的混合不同ClassLoader载入的类，这破坏了由ClassLoader提供的命名空间隔离，会可能引发类型安全问题。
在 JDK/JRE 1.2，每个ClassLoader都有自己的一组本地库。一个本地库一旦被一个ClassLoader加载后，则不允许再被其他ClassLoader重复加载了。否则会抛出 ``UnsatisfiedLinkError 异常。这样的好处是：
	•	由ClassLoader创建的命名空间隔离在本地库也被保存下来了。一个本地库不能轻易混合不同ClassLoader加载的类。
	•	另外，本地库可以在ClassLoader被垃圾收回时unload。
ClassLoader什么时候会被垃圾收回？

JNI_OnLoad
Java虚拟机在加载本地库（native library）时（即调用 System.loadLibrary() ）后，在加载本地库到内存之后，会寻找其内部的 JNI_OnLoad 函数，并执行它。这个函数必须返回本地库使用的 JNI版本号。
如果要使用新的JNI函数，则必须返回高版本的JNI版本号。如果本地库不提供 JNI_ONLoad 函数，则虚拟机默认它使用的是 JNI_VERSION_1_1版本。如果返回一个虚拟机不支持的JNI版本号，则本地库不能被加载。

JNI_OnUnload
虚拟机在本地库被垃圾回收前，调用其 JNI_OnUnload 函数。这个函数用来执行一个清理操作。因为函数在不确定的情况下被调用（例如 finalizer），因此开发者应该保守的使用Java虚拟机服务，并避免任何Java回调函数。
注意：JNI_OnLoad 和 JNI_OnUnload 两个函数是可选的，不是必须要有。

5.3 Invocation API函数
JavaVM类型是一个指向 Invocation API 函数表的指针。例如：
typedef const struct JNIInvokeInterface *JavaVM;

const struct JNIInvokeInterface ... = {
    NULL,
    NULL,
    NULL,

    DestroyJavaVM,
    AttachCurrentThread,
    DetachCurrentThread,

    GetEnv,

    AttachCurrentThreadAsDaemon
};

JNI_GetDefaultJavaVMInitArgs
jint JNI_GetDefaultJavaVMInitArgs(void *vm_args);
返回Java虚拟机的默认配置。
参数：
	•	vm_args ：JavaVMIntArgs结构体的指针，包含虚拟机默认配置。
返回值：
成功返回 JNI_OK ，失败返回负数。

JNI_GetCreatedJavaVMs
jint JNI_CreateJavaVM(JavaVM **p_vm, void **p_env, void *vm_args);
返回所有已经创建过的Java虚拟机。
在 JDK/JRE 1.2, 不支持一个进程创建多个Java虚拟机。
参数：
	•	vmBuf：虚拟机buffer，创建过的虚拟机会被放进来。
	•	bufLen：buffer的最大长度。
	•	nVMs: 虚拟机的总数。
返回值：
成功返回 JNI_OK ，失败返回负数。

JNI_CreateJavaVM
jint JNI_CreateJavaVM(JavaVM **p_vm, void **p_env, void *vm_args);
加载和初始化一个Java虚拟机，当前线程作为主线程（main thread）。
在 JDK/JRE 1.2，不允许在同一个进程创建多个Java虚拟机。
参数：
	•	p_vm：指向 JavaVM的指针。 
	•	p_env：指向 JNIEnv指针的指针。 
	•	vm_args: 虚拟机的参数。  
第3个参数 vm_args 的结构体为：
typedef struct JavaVMInitArgs {
    jint version;

    jint nOptions;
    JavaVMOption *options;
    jboolean ignoreUnrecognized;
} JavaVMInitArgs;
其中 version 必须大于等于 JNI_VERSION_1_2,
其中 nOptions 为 options 的数量.
其中 ignoreUnrecognized 设置为 JNI_TRUE ，则会忽视所有不被识别的以 -X 或 _ 开头的参数字符串，如果设置为 JNI_FALSE ，则遇到不被识别的参数时JNI_CreateJavaVM 函数会返回 JNI_ERR
其中 options 的结构体为：
typedef struct JavaVMOption {
    char *optionString;  /* the option as a string in the default platform encoding */
    void *extraInfo;
} JavaVMOption;
另外，所有虚拟机的实现都支持它自己的非标准参数。非标准参数必须以 -X 或 _ 开头。例如，JDK/JRE 支持 -Xms 和 -Xmx 参数来允许开发者指定初始化和最大的heap大小。
返回值：
成功返回 JNI_OK ，失败返回负数。
使用实例：
JavaVMInitArgs vm_args;
JavaVMOption options[4];

options[0].optionString = "-Djava.compiler=NONE";           /* disable JIT */
options[1].optionString = "-Djava.class.path=c:\myclasses"; /* user classes */
options[2].optionString = "-Djava.library.path=c:\mylibs";  /* set native library path */
options[3].optionString = "-verbose:jni";                   /* print JNI-related messages */

vm_args.version = JNI_VERSION_1_2;
vm_args.options = options;
vm_args.nOptions = 4;
vm_args.ignoreUnrecognized = TRUE;

/* Note that in the JDK/JRE, there is no longer any need to call
 * JNI_GetDefaultJavaVMInitArgs.
 */
res = JNI_CreateJavaVM(&vm, (void **)&env, &vm_args);
if (res < 0) ...

DestoryJavaVM
jint DestroyJavaVM(JavaVM *vm);
卸载一个Java虚拟机，并收回它拥有的资源。
JDK/JRE 1.1 还没有完全支持这个函数。在JDK/JRE 1.1 只有主线程才允许调用该函数。自从 JDK/JRE 1.2，任何线程，不管是否已经 attached，都可以调用该函数，如果当前线程已经 attached，则虚拟机会等待当前线程作为唯一的非守护用户线程。如果当前线程没有 attached，则先attached，再等待当前线程作为唯一的非守护用户线程。
JDK/JRE 1.1.2 不支持unload虚拟机。
参数：
	•	vm：需要被销毁的虚拟机。
返回值：
成功返回 JNI_OK ，失败返回负数。

AttachCurrentThread
jint AttachCurrentThread(JavaVM *vm, void **p_env, void *thr_args);
attach当前线程到Java虚拟机，返回JNI接口指针 JNIEnv 。
尝试attach已经attached过的线程不会执行任何操作（no-op）。
一个本地线程不能同时attach到两个不同的Java虚拟机。
当前一个线程attach到虚拟机，它的上下文ClassLoader是Bootstrap ClassLoader。
参数：
	•	vm：需要被attach到的虚拟机。
	•	p_env ：返回的当前线程的JNI接口指针。
	•	thr_args ：JavaVMAttachArgs 结构体来指定附加信息，或传入 NULL 
typedef struct JavaVMAttachArgs {
    jint version;  /* must be at least JNI_VERSION_1_2 */
    char *name;    /* the name of the thread as a modified UTF-8 string, or NULL */
    jobject group; /* global ref of a ThreadGroup object, or NULL */
} JavaVMAttachArgs
返回值：
成功返回 JNI_OK ，失败返回负数。

AttachCurrentThreadAsDaemon
jint AttachCurrentThreadAsDaemon(JavaVM* vm, void** penv, void* args);
和 AttachCurrentThread 类似，只是新创建的 java.lang.Thread 被设置为守护线程（daemon）。
JDK/JRE 1.4之后才有这个函数。
参数：
	•	vm：需要被attach到的虚拟机。
	•	penv ：返回的当前线程的JNI接口指针。
	•	args ：JavaVMAttachArgs 结构体来指定附加信息，或传入 NULL 
返回值：
成功返回 JNI_OK ，失败返回负数。

DetachCurrentThread
从java虚拟机detach当前线程。所有这个线程持有的Java监视区(monitor)都会被释放。
自从 JDK/JRE 1.2, 主线程可以从虚拟机detach。
参数：
	•	vm：需要detach的虚拟机。
返回值：
成功返回 JNI_OK ，失败返回负数。

GetEnv
jint GetEnv(JavaVM *vm, void **env, jint version);
获取当前线程的JNI接口指针 JNIEnv
参数：
	•	vm：虚拟机实例。
	•	env：放置返回的当前线程的JNI接口指针。
	•	version：JNI版本。
返回值：
如果当前线程还没有attach到虚拟机，则设置 *env 为 NULL ，并返回 JNI_EDETACHED 。如果指定的JNI版本不被支持，则也设置 *env 为 NULL ，并且返回 JNI_EVERSION。否则设置 *env 为正常的接口，并返回 JNI_OK 。

结语
JNI作为Java虚拟机的接口，衔接了Java层和native层。并且涉及到ClassLoader和JVM的内部实现等知识，全面了解JNI可以作为一把钥匙来打开JDK和JVM底层研究的大门。


参考资料：
	•	Java Native Interface官方技术手册
	•	Google JNI官方文档
	•	JNI教程与技术手册
	•	http://www.kancloud.cn/owenoranba/jni/120449
	•	JNI官方规范中文版
	•	Java本地接口(WIKI)
	•	Dalvik虚拟机JNI方法的注册过程分析
	•	JNI函数
	•	jni.h头详见详解二
	•	OpenJDK源码 /openjdk/hotspot/src/share/vm/prims/jni.h
	•	OpenJDK源码 /openjdk/hotspot/src/share/vm/prims/jni.c
	•	Android JNI编程提高篇之二
	•	JNI引用于垃圾回收
	•	http://blog.csdn.net/autumn20080101/article/details/8646431
	•	C++访问Java的String字符串对象


作者：骆驼骑士
链接：https://www.jianshu.com/p/37694530f97c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

