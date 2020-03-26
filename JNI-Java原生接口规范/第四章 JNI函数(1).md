第四章 JNI函数
本章作为JNI函数的参考手册，提供了完整的JNI函数列表，及每个函数的文档说明。
JNI开发者一定要注意标记为must的限制。例如，当你看到一个JNI函数表示 must 接受一个不为null的对象，name你就有义务一定不要传null给该JNI函数，因为JNI函数在内部并不会检查空指针，你需要在调用之前自己检查。
这一章节的部分内容和网景提出的JRI文档是兼容的。

注意：因为官方文档中对于每个函数都没有举出例子说明，为了便于理解，因此本文作者会尽量对每个函数找出使用的实例来便于理解。请注意这些实例并非官方文档的一部分。

4.1 接口函数表
所有的JNI函数都是通过传递进来固定的第一个参数 JNIEnv 参数来进行访问到的。JNIEnv 类型是一个指向所有JNI函数指针集合的指针。它的定义如下：
typedef const struct JNINativeInteface *JNIEnv;
虚拟机初始化函数表如下，前三个reserved值是为了兼容COM标准：
const struct JNINativeInterface_ {

    NULL,
    NULL,
    NULL,
    NULL,
    GetVersion,

    DefineClass,
    FindClass,

    FromReflectedMethod,
    FromReflectedField,
    ToReflectedMethod,

    // 后面的省略掉，函数太多了。
};

4.2 版本信息
GetVersion
jint GetVersion(JNIEnv *env);
返回JNI的版本号。
参数：
	•	env: jni接口指针
返回值：
返回一个值，其中高位为major版本号返回，低位为minor版本号。
在 JDK/JRE 1.1中返回 0x00010001
在 JDK/JRE 1.2中返回 0x00010002
在 JDK/JRE 1.4中返回 0x00010004
在 JDK/JRE 1.6中返回 0x00010006
常量：
SINCE JDK/JRE 1.2:
#define JNI_VERSION_1_1 0x00010001
#define JNI_VERSION_1_2 0x00010002

/* Error codes */
#define JNI_EDETACHED    (-2)         /* thread detached from the VM */
#define JNI_EVERSION     (-3)         /* JNI version error 
SINCE JDK/JRE 1.4:
#define JNI_VERSION_1_4 0x00010004
SINCE JDK/JRE 1.6:
#define JNI_VERSION_1_6 0x00010006
使用实例：
jint version = env->GetVersion();

4.3 操作类
DefineClass
jclass DefineClass(JNIEnv *env, const char *name, jobject loader,
                  const jbyte *buf, jsize bufLen);
题外话：请注意这个函数不会Android支持，因为Android并不使用Java字节码或class文件，所以传入二进制的class data不会起作用。
从包含二进制的类数据(binary class data)的buffer中载入类。
在调用DefineClass完毕后，虚拟机不会再引用这个buffer，你可以释放掉它。
参数：
	•	env：JNI接口指针
	•	name：需要加载的类或接口的短名字，这个字符串使用MUTF-8编码。
	•	loader：指派用来加载类的ClassLoader
	•	buf：包含 .class 文件数据的 buffer
	•	bufLen: buffer的长度
返回值：
返回加载的Java类对象（java class object），或者如果出现错误则返回NULL
抛出异常：
ClassFormatError ：如果传入的class内容不是一个有效的class文件。
ClassCircularityError：如果class或interface是它自己的父类或父接口，造成循环层级关系。
OutOfMemoryError：如果系统在载入的过程中内存不足。
SecurityException ：如果调用者启动载入java包内置的类，引发安全隐患。
使用实例：
// Read in file from temp source
HANDLE hFile = CreateFile(filename, GENERIC_READ, FILE_SHARE_READ, NULL,  
                          OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
DWORD cbBuffer = GetFileSize(hFile, 0);
PBYTE pBuffer = (PBYTE) malloc(cbBuffer);
ReadFile(hFile, pBuffer, cbBuffer, &cbBuffer, 0);
CloseHandle(hFile);

// DefineClass
jclass cl = env->DefineClass(name, loader, (const jbyte*) pBuffer, cbBuffer);

free(pBuffer);
以上代码来至：https://github.com/poidasmith/winrun4j/blob/e4ab3564c7a580a9d4801c5954aacda667f88b3b/WinRun4J/src/java/JNI.cpp

FindClass
jclass FindClass(JNIEnv *env, const char *name);
在JDK 1.1发行版中，这个函数加载本地定义的类（locally-defined class）, 它搜索在由环境变量 CLASSPATH 目录下的子目录和zip文件中搜索指定的类名。
从JDK 1.2发行版之后，Java安全模型允许非本地系统类也可以加载和调用本地方法。FindClass 函数会使用与当前本地方法关联的ClassLoader, 并用它来加载本地方法指定的class。如果本地方法属于系统类（system class），则没有ClassLoader会被调用。否则，将使用正确的ClassLoader来加载(load)和链接(link)指定名称的类。
从JDK 1.2发行版之后，当通过 Invocation 接口来调用 FindClass 函数，将没有当前的本地方法或与之关联的ClassLoader。这种情况下，会使用 ClassLoader.getSystemClassLoader 来替代。这个ClassLoader 是虚拟机用来创建应用(applications)的，它有能力定位到 java.class.path 参数下的所有类。
第二个 name 参数，使用全称类名或数组类型签名（array type signature）。例如，String类的全称类名为：
"java/lang/String"
而Object数组类型的使用：
"[Ljava/lang/Object;"
题外话：一定要注意分隔符不是 . 而是 / ，不要写错了。
参数：
	•	env：JNI接口指针
	•	name：全称的类名（包名以 / 作为分隔符, 然后紧跟着类名），如果名字以 [开头（数组签名标识符），则返回一个数组的类，这个字符串也是MUTF-8。
返回值：
指定名称的类的对象（a class object），或者在没有找到对应类时返回 NULL
抛出异常：
ClassFormatError ：如果class内容不是一个有效的class文件。
ClassCircularityError：如果class或interface是它自己的父类或父接口，造成循环层级关系。
OutOfMemoryError：如果系统在载入的过程中内存不足。
NoClassDefFoundError：如果指定的类或接口没有被找到。（当name传null或超长时也会抛出这个异常）
题外话：关于Java安全模型更多信息可参考：https://www.ibm.com/developerworks/cn/java/j-lo-javasecurity/index.html
使用实例：
 // Start thread which receives commands from the SA.
jclass threadClass = env->FindClass("java/lang/Thread");
if (threadClass == NULL) stop("Unable to find class java/lang/Thread");

jstring threadName = env->NewStringUTF("Serviceability Agent Command Thread");
if (threadName == NULL) stop("Unable to allocate debug thread name");

jmethodID ctor = env->GetMethodID(threadClass, "<init>", "(Ljava/lang/String;)V");
if (ctor == NULL) stop("Unable to find appropriate constructor for java/lang/Thread");

// Allocate thread object
jthread thr = (jthread) env->NewObject(threadClass, ctor, threadName);   
以上代码来之 openJDK 源码 中的 /hotspot/agent/src/share/native/jvmdi/sa.cpp

GetSuperclass
jclass GetSuperclass(JNIEnv *env, jclass clazz);
只要传入的 clazz 参数不是 java/lang/Object 则返回该类的父类。
如果传入的 clazz 参数是 java/lang/Object 则返回NULL，因为它没有父类。当传入的是一个接口，而不是类时，也返回 NULL 。
参数：
	•	env：JNI接口指针
	•	clazz： Java类对象（java class object）
返回值：
返回传入的 clazz 的父类，或 NULL .
使用实例：
jclass clazz = env->GetObjectClass(thiz);

clazz = env->GetSuperclass(clazz);

jfieldID __state = env->GetFieldID(clazz, "__state", "J");
以上代码来至：https://github.com/liucheng98/mesos-0.22.0/blob/5cfd2c36c0e8361fa2e2d9b6f191a738d22a9cfd/src/java/jni/org_apache_mesos_state_LogState.cpp

IsAssignableForm
jboolean IsAssignableFrom(JNIEnv *env, jclass class1, jclass clazz2);
检查 clazz1 的对象是否能被安全的转型（cast）为 clazz2
参数：
	•	env：JNI接口指针
	•	clazz1：第一个class参数（需要转型的类）
	•	clazz2：第二个class参数（转型的目标类）
返回值：
如果是以下情况则返回 JNI_TRUE :
	•	clazz1 和 clazz2 指向同一个java类
	•	clazz1 是 clazz2 的子类。（向上转型是安全的）
	•	clazz1 是 clazz2（接口）的实现类。（也属于向上转型）
使用实例：
jclass listInterface = env->FindClass("java/util/List")
jclass arrayListClass = env->FindClass("java/util/ArrayList")
jboolean isSafe = env->IsAssignableFrom(arrayListClass, listInterface);
// isSafe: true;
isSafe = env->IsAssignableFrom(listInterface, arrayListClass);
// isSafe: false;

4.4 异常
Throw
jint Throw(JNIEnv *env, jthrowable obj);
触发一个 java.lang.Throwable 对象的异常被抛出。
参数：
	•	env：JNI接口指针
	•	obj： java.lang.Throwable 对象
返回值：
成功则返回0， 失败时返回赋值
抛出异常：
抛出 java.lang.Throwable 对象
使用实例：
jthrowable exception = env->ExceptionOccurred();
if (exception) {
  env->ExceptionClear();
  detach_internal(env, this_obj);
  env->Throw(exception);
  return;
}
以上代码来至 OpenJDK 源码 /hotspot/agent/src/os/solaris/proc/saproc.cpp

ThrowNew
jint ThrowNew(JNIEnv *env, jclass clazz, const char *message);
Exception对象的构造器函数，message为异常的错误消息，clazz为异常的类。
参数：
	•	env：JNI接口指针
	•	clazz：java.lang.Throwable 的子类
	•	message： 用于创建 java.lang.Throwable 对象时传入的错误消息。这个是字符串是MUTF-8编码。
返回值：
成功则返回0， 失败时返回赋值
抛出异常：
抛出刚构造出来的 java.lang.Throwable 对象
使用实例：
env->ThrowNew(env->FindClass("sun/jvm/hotspot/debugger/DebuggerException"), errMsg);

ExceptionOccurred
jthrowable ExceptionOccurred(JNIEnv *env);
检查是否有异常被抛出。这个异常在本地方法调用 ExceptionClear() 方法或被Java代码处理这个异常之前都会保持在被抛出状态。
参数：
	•	env：JNI接口指针
返回值：
返回过程中抛出的异常，或没有异常被抛出时返回 NULL 。
使用实例：
jthrowable exception = env->ExceptionOccurred();
if (exception) {
  env->ExceptionClear();
  detach_internal(env, this_obj);
  env->Throw(exception);
  return;
}
以上代码来至 OpenJDK 源码 /hotspot/agent/src/os/solaris/proc/saproc.cpp

ExceptionDescribe
void ExceptionDescribe(JNIEnv *env);
打印一个异常的stack trace到系统的错误输出，例如 stderr 这是为了调试提供便利。
参数：
	•	env：JNI接口指针
使用实例：
jthrowable exc = safe_ExceptionOccurred(env);
if (exc) {
    env->DeleteLocalRef(exc);
    env->ExceptionDescribe();
    env->ExceptionClear();
}
以上代码来至：OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Object.cpp

ExceptionChar
void ExceptionClear(JNIEnv *env);
清理任何即将抛出的异常。如果没有异常被抛出，则不起任何作用。
参数：
	•	env：JNI接口指针
使用实例：
jthrowable exc = safe_ExceptionOccurred(env);
if (exc) {
    env->DeleteLocalRef(exc);
    env->ExceptionDescribe();
    env->ExceptionClear();
}
以上代码来至：OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Object.cpp

FatalError
void FatalError(JNIEnv *env, const char *msg);
抛出一个严重错误，并不希望虚拟机恢复。
参数：
	•	env：JNI接口指针
	•	msg ： 错误消息，这个字符串为MUTF-8编码
使用实例：
env->CallVoidMethod(currentThread, setThreadName, threadName) ;
if( env->ExceptionCheck() ) {
  env->ExceptionDescribe() ;
  env->FatalError("setting thread name failed: could not start reading thread");
  return 0L ;
}
以上代码来至：https://github.com/tnarnold/netsnmpj/blob/3153c3e9297dd7de7db9d1a65c97f77937ae147b/netsnmpj-preliminary/native/nativeThread.cc

ExceptionCheck
jboolean ExceptionCheck(JNIEnv *env);
这是一个快速函数用于检查是否有被抛出的异常，而不创建一个这个异常的局部引用。
参数：
	•	env：JNI接口指针
返回值：
有一个即将被抛出的异常时返回 JNI_TURE ，没有则返回 JNI_FALSE
使用实例：
env->CallVoidMethod(thread, runId);

if (env->ExceptionCheck()) {
  env->ExceptionDescribe();
  env->ExceptionClear();
  // handle exception
}
以上代码来至：OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Toolkit.cpp


作者：骆驼骑士
链接：https://www.jianshu.com/p/6b80903ef191
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


