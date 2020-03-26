第四章 JNI函数
4.15 操作监视器(同步锁)
MonitorEnter
jint MonitorEnter(JNIEnv *env, jobject obj);
进入一个obj的监视区monitor。
即使用 obj 作为锁对象（可能是对象锁，也可能是类锁），obj必须不能为 null。
每个java对象内部都有一个锁（monitor），如果当前线程已经拥有了obj的monitor，它会在其监视区增加计数器，记录线程进入监视区的次数。如果obj上的监视区没有被任何其他线程持有，则当前线程开始持有监视区的锁，设置监视区的计数器为1；如果其他线程已经拥有了这个监视区的锁，则当前线程一直等待锁被释放，然后再尝试获取锁。
通过 MonitorEnter 这个JNI函数进入到一个监视区中，是无法使用Java虚拟机 monitorexit 指令或一个同步方法返回来退出这个监视区的。MonitorEnter函数和Java虚拟机 monitorexit 指令之间可能抢着进入同一个obj的监视区。
为了避免死锁，使用 MonitorEnter方法来进入监视区，必须使用MonitorExit 来退出，或调用 DetachCurrentThread 来明确释放JNI监视区。
参数：
	•	env ：JNI接口指针
	•	obj：一个普通的java对象或class对象。
返回值：
成功返回0，失败返回负数。
使用实例：
env->MonitorEnter(mMonitor);
env->CallVoidMethod(mMonitor, mWaitMethod);
env->MonitorExit(mMonitor);
以上代码来至：https://github.com/abhijit86k/iceweasel/blob/140d9f57f5e4240bec0545b5a650f8ef9d3f45d3/plugin/oji/MRJ/plugin/Source/MRJMonitor.cpp

MonitorExit
jint MonitorExit(JNIEnv *env, jobject obj);
当前线程必须持有obj的monitor，当前线程进入monitor的计数器减1，如果计数器的值变为了0，则当前线程释放monitor。
本地代码不能使用 MonitorExit 函数来退出一个通过Java虚拟机 monitorexit指令或一个同步方法进入的monitor。
参数：
	•	env ：JNI接口指针
	•	obj：一个java对象或一个class对象
返回值：
成功返回0，失败返回负数。
排除异常：
	•	IllegalMonitorStateException如果当前线程还没持有monitor。


4.16 NIO支持
NewDirectByteBuffer
jobject NewDirectByteBuffer(JNIEnv* env, void* address, jlong capacity);
分配并返回一个 java.nio.ByteBuffer ，指向一块内存，地址开始于 address ，并有 capacity 个bytes的内存空间。
本地代码调用这个函数并生成byte-buffer对象给一个Java层，必须确保这个buffer指向一个有效的可读内存空间，如果需要，也得是可写的内存。如果从Java代码尝试访问一个无效的内存空间，可能返回一个随机的值，或没有任何可见反应，或抛出一个未指定的异常。
JDK/JRE 1.4及以上支持该函数。
参数：
	•	env ：JNI接口指针
	•	address：内存空间起始地址（必须不为 NULL )
	•	capacity：内存空间的大小（字节数）（必须为整数）
返回值：
返回刚创建的 java.nio.ByteBuffer 对象的局部引用。如果有异常抛出或者虚拟机不允许JNI访问直接的buffer，则返回 NULL
抛出异常：
	•	OutOfMemoryError: 如果分配 ByteBuffer 对象失败时。
使用实例：
  jobject pDataBuf = null;
  if (filep->data[0].len > 0)
    pDataBuf = env->NewDirectByteBuffer(filep->data[0].ptr,
                                        filep->data[0].len);
  env->SetObjectArrayElement(pParts, pidx++, pDataBuf);
  pDataBuf = null;
  if (filep->data[1].len > 0)
    pDataBuf = env->NewDirectByteBuffer(filep->data[1].ptr,
                                        filep->data[1].len);
  env->SetObjectArrayElement(pParts, pidx++, pDataBuf);
以上代码来至：Jni.cpp (openjdk\jdk\src\share\native\com\sun\java\util\jar\pack)

GetDirectBufferAddress
void* GetDirectBufferAddress(JNIEnv* env, jobject buf);
获取给定的 java.nio.Buffer 的起始内存地址。
这个函数通过buffer对象允许本地代码访问Java代码可访问的同一块内存地址。
JDK/JRE 1.4及以上支持该函数。
参数：
	•	env ：JNI接口指针
	•	buf：给定的 java.nio.Buffer 对象（必须不能为 NULL )
返回值：
返回buffer指向的起始内存地址。如果内存地址是undefined的，如果给定的jobject不是 java.nio.Buffer 或者如果虚拟机不支持JNI访问直接的buffer，则返回 NULL 。
使用实例：
JNIEXPORT jclass JNICALL
Java_java_lang_ClassLoader_defineClass2(JNIEnv *env,
                                        jobject loader,
                                        jstring name,
                                        jobject data,
                                        jint offset,
                                        jint length,
                                        jobject pd,
                                        jstring source)
{
    jbyte *body;
    char *utfName;
    jclass result = 0;
    char buf[128];
    char* utfSource;
    char sourceBuf[1024];

    assert(data != NULL); // caller fails if data is null.
    assert(length >= 0);  // caller passes ByteBuffer.remaining() for length, so never neg.
    // caller passes ByteBuffer.position() for offset, and capacity() >= position() + remaining()
    assert((*env)->GetDirectBufferCapacity(env, data) >= (offset + length));
  
    body = (*env)->GetDirectBufferAddress(env, data);

    if (body == 0) {
        JNU_ThrowNullPointerException(env, 0);
        return 0;
    }
  
    // ...
}
以上代码来至：ClassLoader.c (openjdk\jdk\src\share\native\java\lang)

GetDirectBufferCapacity
jlong GetDirectBufferCapacity(JNIEnv* env, jobject buf);
获取并返回给定 java.nio.Buffer 对象的内存空间大小（capacity）。capacity是内存空间包含的元素的个数（the number of elements that the memory region contains）
参数：
	•	env ：JNI接口指针
	•	buf : 给定的buffer对象。（必须不能为 NULL )
返回值：
返回buffer对应的内存空间的capacity。如果给定的jobject不是 java.nio.Buffer ，或者the object is an unaligned view buffer and the processor architecture does not support unaligned access，或者虚拟机不允许JNI访问direct buffer，则返回 -1 。
使用实例：
请参见上一个函数的例子。


4.17 反射支持
开发者如果知道方法或域的名称和类型，可以使用JNI来调用Java方法或访问Java域。Java反射API提供运行时自省。JNI提供一组函数来使用Java反射API。

FromReflectedMethod
jmethodID FromReflectedMethod(JNIEnv *env, jobject method);
通过 java.lang.reflect.Method 或 java.lang.reflect.Constructor 来获取其反射的目标方法对应的 methodId.
参数：
	•	env ：JNI接口指针
	•	jobject：一个 java.lang.reflect.Method 或 java.lang.reflect.Constructor 对象。
返回值：
反射的目标方法的methodID
使用实例：
extern void __attribute__ ((visibility ("hidden"))) dalvik_replaceMethod(
        JNIEnv* env, jobject src, jobject dest) {
    jobject clazz = env->CallObjectMethod(dest, jClassMethod);
    ClassObject* clz = (ClassObject*) dvmDecodeIndirectRef_fnPtr(
            dvmThreadSelf_fnPtr(), clazz);
    clz->status = CLASS_INITIALIZED;

    Method* meth = (Method*) env->FromReflectedMethod(src);
    Method* target = (Method*) env->FromReflectedMethod(dest);
    LOGD("dalvikMethod: %s", meth->name);

//  meth->clazz = target->clazz;
    meth->accessFlags |= ACC_PUBLIC;
    meth->methodIndex = target->methodIndex;
    meth->jniArgInfo = target->jniArgInfo;
    meth->registersSize = target->registersSize;
    meth->outsSize = target->outsSize;
    meth->insSize = target->insSize;

    meth->prototype = target->prototype;
    meth->insns = target->insns;
    meth->nativeFunc = target->nativeFunc;
}
以上代码来至：https://github.com/alibaba/AndFix/blob/67a5e3c2c308569f39400cc3258545ee25538719/jni/dalvik/dalvik_method_replace.cpp
以下是个人理解部分:
这里讲一个题外话，上面的例子是来至阿里巴巴开源的Android热修复框架andFix。其中就使用了 FromReflectedMethod 来实现替换有BUG的方法，以实现热修复的目标。
它的实现原理是通过将目标方法修改为native方法，然后修改其 nativeFunc 的指针来实现替换一个方法。

FromReflectedField
jfieldID FromReflectedField(JNIEnv *env, jobject field);
通过 java.lang.reflect.Field 对象来获取fieldID.
参数：
	•	env ：JNI接口指针
	•	field：java.lang.reflect.Field 对象
返回值：
目标成员域的fieldID
使用实例：
JNIEXPORT void JNICALL Java_brian_com_nativehotfixdemo_hotfix_HotfixManager_setSingleFlagArt
        (JNIEnv* env, jobject obj, jobject field) {
    Field* dalvikField = (Field*) env->FromReflectedField(field);
    dalvikField->accessFlags = dalvikField->accessFlags & (~ACC_PRIVATE) | ACC_PUBLIC;
}
以上代码来至：https://github.com/brianxcli/NativeHotFixDemo/blob/b9e6399ca7ab18608bab69df62328cfa9223f39f/app/src/main/cpp/hotfix_dalvik.cpp

ToReflectedMethod
jobject ToReflectedMethod(JNIEnv *env, jclass cls, jmethodID methodID, jboolean isStatic);
转换 cls 的 methodID 为 java.lang.reflect.Method 或java.lang.reflect.Constructor对象。
参数：
	•	env ：JNI接口指针
	•	cls：目标方法的java类对象
	•	jmethodId :目标方法的methodID
	•	isStatic：目标方法为静态的则比较将 isStatic 设置为 JNI_TRUE ，否则设置为 JNI_FALSE 。
返回值：
java.lang.reflect.Method 或java.lang.reflect.Constructor对象
使用实例：
extern "C" JNIEXPORT jobject JNICALL Java_JniTest_testGetMirandaMethodNative(JNIEnv* env, jclass) {
  jclass abstract_class = env->FindClass("JniTest$testGetMirandaMethod_MirandaAbstract");
  assert(abstract_class != NULL);
  jmethodID miranda_method = env->GetMethodID(abstract_class, "inInterface", "()Z");
  assert(miranda_method != NULL);
  return env->ToReflectedMethod(abstract_class, miranda_method, JNI_FALSE);
}
以上代码来至：https://github.com/android-security/android_art/blob/479fe13d929012cd45c7a92ce140dae0b71e026f/test/JniTest/jni_test.cc

ToReflectedField
jobject ToReflectedField(JNIEnv *env, jclass cls, jfieldID fieldID, jboolean isStatic);
转换 cls 的 fieldID 为 java.lang.reflect.Field 对象。
参数：
	•	env ：JNI接口指针
	•	cls：目标方法的java类对象
	•	jmethodId :目标方法的methodID
	•	isStatic：目标方法为静态的则比较将 isStatic 设置为 JNI_TRUE ，否则设置为 JNI_FALSE 。
返回值：
java.lang.reflect.Field 对象


4.18 Java虚拟机接口
GetJavaVM
jint GetJavaVM(JNIEnv *env, JavaVM **vm);
返回关联到当前线程的Java虚拟机接口指针。
参数：
	•	env ：JNI接口指针
	•	vm: 用于存放返回的Java虚拟机指针
返回值：
成功返回0，失败返回负数。
使用实例：
if (JNI_OnLoad) {
  JavaVM *jvm;
  (*env)->GetJavaVM(env, &jvm);
  jniVersion = (*JNI_OnLoad)(jvm, NULL);
} else {
  jniVersion = 0x00010001;
}
以上代码来至：OpenJDK/jdk/src/share/native/java/lang/ClassLoader.c



作者：骆驼骑士
链接：https://www.jianshu.com/p/c29862d581be
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


