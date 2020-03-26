第四章 JNI函数
4.5 全局和局部引用
局部引用只在本地方法执行的过程有有效。在本地方法返回时它们会被自动释放掉。所有的局部引用都有消耗一些Java虚拟机的资源。开发者必须确保本地方法不会过度的使用局部引用，尽管局部引用会在本地方法返回到Java后自动释放，但过度的使用可能会引发虚拟机在执行本地方法时内存不足。
NewGlobalRef
jobject NewGlobalRef(JNIEnv *env, jobject obj);
创建一个 obj 参数对象的全局引用
参数：
	•	env ：JNI接口指针
	•	obj：一个全局或本地的对象引用
返回值：
返回一个全局引用，或者当系统内存不足时返回 NULL
使用实例：
// Remember which thread this is
debugThreadObj = env->NewGlobalRef(thr);
if (debugThreadObj == NULL) stop("Unable to allocate global ref for debug thread object");
以上代码来至：OpenJDK 源码中的 /hotspot/agent/src/share/native/jvmdi/sa.cpp

DeleteGlobalRef
void DeleteGlobalRef(JNIEnv *env, jobject globalRef);
删除指向 gloablRef 的全局变量。
参数：
	•	env ：JNI接口指针
	•	globalRef: 需要删除的全局引用
返回值：
无
使用实例：
jthread thr = suspendedThreads[i];
jvmdiError err = jvmdi->ResumeThread(thr);
env->DeleteGlobalRef(thr);
以上代码来至：OpenJDK 源码中的 /hotspot/agent/src/share/native/jvmdi/sa.cpp

DeleteLocalRef
void DeleteLocalRef(JNIEnv *env, jobject localRef);
删除 localRef 的局部引用。
参数：
	•	env ：JNI接口指针
	•	localRef：需要删除的局部引用。
返回值：
无
使用实例：
for (i = 0; i < itemCount; i++)
{
  jstring item = (jstring)env->GetObjectArrayElement(items, i);
  JNI_CHECK_NULL_GOTO(item, "null item", next_elem);
  c->SendMessage(CB_INSERTSTRING, index + i, JavaStringBuffer(env, item));
  env->DeleteLocalRef(item);
}
以上代码来至：OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Choice.cpp
注意：
在JDK/JRE 1.1的时候提供 DeleteLocalRef 函数，因此开发者可以手动的删除局部引用。例如，在本地方法遍历一个大数组的时候，需要在遍历过程中对每个元素进行处理，在这种情况下正确的做法是在创建一个新的本地引用之前，先把上一个不再被使用的本地引用删除掉。
而在JDK/JRE 1.2 增加一个一组以下4个函数来提供管理局部引用的生命周期。

EnsureLocalCapacity
jint EnsureLocalCapacity(JNIEnv *env, jint capacity);
确保在当前线程中至少还有 capacity 个本地引用可能被创建。
在每进入一个本地方法前，Java虚拟机自动确保至少可以上传 16 个局部引用。
为了向后兼容，虚拟机在超过 capacity 之后还是可以允许创建本地引用（为了方便调试，虚拟机会在太多局部引用被创建时会给用户警告。在JDK里，开发者可以使用 -verbose:jni 命令行参数来开启这些警告消息）。虚拟机会在无法创建局部引用时抛出 FatalError 。
参数：
	•	env ：JNI接口指针
	•	capacity:
返回值：
返回 0 表示还可以创建；否则返回一个负数，并抛出 OutOfMemoryError 。
使用实例：
if (env->EnsureLocalCapacity(1) < 0) {
  return NULL;
}
以上代码来至：OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Canvas.cpp
题外话：虚拟机对于capacity的上限在源码中定义为常量：
// Pick a reasonable higher bound for local capacity requested
// for EnsureLocalCapacity and PushLocalFrame.  We don't want it too
// high because a test (or very unusual application) may try to allocate
// that many handles and run out of swap space.  An implementation is
// permitted to allocate more handles than the ensured capacity, so this
// value is set high enough to prevent compatibility problems.
const int MAX_REASONABLE_LOCAL_CAPACITY = 4*K;

PushLocalFrame
jint PushLocalFrame(JNIEnv *env, jint capacity);
创建一个局部引用的栈帧，其可以创建最多 capacity 个局部引用。
注意在之前栈帧中已经创建的局部引用在当前栈帧中依然有效。
参数：
	•	env ：JNI接口指针
	•	capacity：最多可创建的局部引用数
返回值：
成功创建则返回0，失败则返回一个负数，并抛出 OutOfMenoryError 。
使用实例：
    if (env->PushLocalFrame(1) < 0)
        return;

    jobject target = GetTarget(env);

    // To avoid possibly running client code on the toolkit thread, don't
    // do the following checks if we're running on the toolkit thread.
    if (AwtToolkit::MainThread() != ::GetCurrentThreadId()) {
        // Compare number of items.
        int nTargetItems = JNU_CallMethodByName(env, NULL, target,
                                                "countItems", "()I").i;
        DASSERT(!safe_ExceptionOccurred(env));
        int nPeerItems = (int)::SendMessage(GetHWnd(), CB_GETCOUNT, 0, 0);
        DASSERT(nTargetItems == nPeerItems);

        // Compare selection
        int targetIndex = JNU_CallMethodByName(env, NULL, target,
                                               "getSelectedIndex", "()I").i;
        DASSERT(!safe_ExceptionOccurred(env));
        int peerCurSel = (int)::SendMessage(GetHWnd(), CB_GETCURSEL, 0, 0);
        DASSERT(targetIndex == peerCurSel);
    }

    env->PopLocalFrame(0);
以上代码来至：OpenJDK源码中的 /jdk/src/windows/native/sun/windows/awt_Choice.cpp

PopLocalFrame
jobject PopLocalFrame(JNIEnv *env, jobject result);
将当前栈帧出栈，释放所有局部引用
参数：
	•	env ：JNI接口指针
	•	result: 传入 NULL 表示不需要返回上一栈帧的引用。
返回值：
返回给定 result 对象的上一个本地引用栈帧本地引用（a local reference in the previous local reference frame for given result object)
使用实例：
    if (env->PushLocalFrame(2) < 0) {
        return E_OUTOFMEMORY;
    }

    jbyteArray bytes =
        AwtDataTransferer::ConvertData(env, m_component, m_transferable,
                                       (jlong)matchedFormatEtc.cfFormat,
                                       m_formatMap);
    if (!JNU_IsNull(env, safe_ExceptionOccurred(env))) {
        env->ExceptionDescribe();
        env->ExceptionClear();
        env->PopLocalFrame(NULL);
        return E_UNEXPECTED;
    }
    if (bytes == NULL) {
        env->PopLocalFrame(NULL);
        return E_UNEXPECTED;
    }
以上代码来至：OpenJDK 源码中的 jdk/src/windows/native/sun/windows/awt_DnDDS.cpp

NewLocalRef
jobject NewLocalRef(JNIEnv *env, jobject ref);
创建一个局部引用同样指向 ref。
参数：
	•	env ：JNI接口指针
	•	ref：可能是一个全局引用，或一个局部引用.
返回值：
返回创建的局部引用，或者如果ref传入null，则返回 NULL
使用实例：
    jobject localObj = env->NewLocalRef(jCursor);
    if (localObj != NULL) {
        setPData(localObj, ptr_to_jlong(NULL));
        env->DeleteLocalRef(localObj);
    }
    env->DeleteWeakGlobalRef(jCursor);
以上代码来至：OpenJDK源码中的 /jdk/src/windows/native/sun/windows/awt_Cursor.cpp


4.6 弱全局引用
弱全局引用是一种特殊的全局引用。和普通全局引用不同的是，一个弱全局引用允许其指向的Java对象可以被垃圾回收。弱全局引用可以在任何可以使用局部引用或全局引用的地方一样的使用。当垃圾回收器开始回收时，它会将只指向软引用的对象回收掉。一个弱全局引用指向的被释放的对象，其功能上相当于 NULL 。开发者可以通过 IsSameObject 来比较一个弱全局引用和 Null 是否相同来检查其是否指针了一个被释放的对象。
弱全局引用是Java弱引用（Java Weak References）的一个简单实现版本，是 Java 2 平台API的一部分（java.lang.ref 包下）。
澄清（2001年6月新加）
因为垃圾回收器可能在本地方法执行的同时进行回收，因此被弱全局引用指向的Java对象随时都可能被释放掉。当弱全局引用被当做全局引用一样使用时，会不是很合适，因为它们可能在没有通知的情况下就变成和 NULL 一样的对象了。
虽然 IsSameObject 可以被用于检查一个弱全局引用是否指向一个被释放的对象。但它并不能防止对象之后立即被释放掉。所以，开发者并不能依赖这种检查来确定在今后的JNI函数调用时这个弱全局对象可用（不是 NULL 引用）。
为了克服这个局限性，建议的方法是使用 NewLocalRef 或 NewGlobalRef 函数来创建一个标准的（强）局部或全局引用来指向同一个对象（同弱全局引用指向的那个对象），这些函数会在对象被释放后返回 NULL ，而没有被释放则返回一个强引用（能防止对象被释放掉）。这个新的引用必须在不需要使用的时候立即明确的删除掉，来允许该对象允许被释放。

NewWeakGlobalRef
jweak NewWeakGlobalRef(JNIEnv *env, jobject obj);
创建一个新的弱全局引用
参数：
	•	env ：JNI接口指针
	•	obj：一个全局引用或局部引用对象。
返回值：
返回刚创建的新弱全局引用，如果obj参数为null或虚拟机内存不足时则返回NULL，如果虚拟机内存不足，还会抛出 OutOfMemoryError
使用实例：
    // jCursor是一个弱引用，在使用前先使用 NewLocalRef 创建一个强引用，必须其指向的对象被释放掉。
    jobject localObj = env->NewLocalRef(jCursor);
    if (localObj != NULL) {
        setPData(localObj, ptr_to_jlong(NULL));
        env->DeleteLocalRef(localObj); // 使用完后，立即释放强引用，使得弱引用指向的对象可以被释放掉
    }
    // 删除弱全局引用
    env->DeleteWeakGlobalRef(jCursor);
以上代码来至：OpenJDK源码中的 /jdk/src/windows/native/sun/windows/awt_Cursor.cpp

DeleteWeakGlobalRef
void DeleteWeakGlobalRef(JNIEnv *env, jweak obj);
删除一个弱引用。
参数：
	•	env ：JNI接口指针
	•	jweak ：需要删除的弱引用
返回值：
无
使用实例：
    // jCursor是一个弱引用，在使用前先使用 NewLocalRef 创建一个强引用，必须其指向的对象被释放掉。
    jobject localObj = env->NewLocalRef(jCursor);
    if (localObj != NULL) {
        setPData(localObj, ptr_to_jlong(NULL));
        env->DeleteLocalRef(localObj); // 使用完后，立即释放强引用，使得弱引用指向的对象可以被释放掉
    }
    // 删除弱全局引用
    env->DeleteWeakGlobalRef(jCursor);
以上代码来至：OpenJDK源码中的 /jdk/src/windows/native/sun/windows/awt_Cursor.cpp

4.7 操作对象
AllocObject
jobject AllocObject(JNIEnv *env, jclass clazz);
在不调用类的任何一个构造器来分配一个新的Java对象。
参数：
	•	env ：JNI接口指针
	•	clazz：需要初始化的类，不能是一个数组类型的类。
返回值：
返回新的尚未初始化的Java对象的引用。如果不能分配对象则返回 NULL
抛出异常：
	•	InstantiationException 如果传入的 clazz 是一个抽象类或接口。
	•	OutOfMemoryError 如果系统没有内存时。
使用实例：
 // 1、创建一个未初始化的对象，并分配内存
 obj_cat = (*env)->AllocObject(env, cls_cat);
 if (obj_cat) {
    // 2、调用对象的构造函数初始化对象
    (*env)->CallNonvirtualVoidMethod(env, obj_cat, cls_cat, mid_cat_init, c_str_name);
    if ((*env)->ExceptionCheck(env)) { // 检查异常
        goto quit;
    }
 }
以上代码来至：http://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/function2.html

NewObject, NewObjectA, NewObjectV
jobject NewObject(JNIEnv *env, jclass clazz, jmethodID methodID, ...);

jobject NewObjectA(JNIEnv *env, jclass clazz, jmethodID methodID, const jvalue *args);

jobject NewObjectV(JNIEnv *env, jclass clazz, jmethodID methodID, va_list args);
构造（初始化）一个新的Java对象，参数methodID指向需要被调用的构造器函数，这个methodID必须使用 GetMethodID() 来获取，并且必须使用 <init> 作为方法名，void(V) 作为返回类型。
clazz 参数必须不指向一个数组类型的类。
NewObject ：
可传入将所有需要传入给构造器的参数放置在 methodID 参数之后，NewObject函数将接受三个参数，并将其传入到Java构造器方法内。
NewObjectA:
可将所有需要传入给构造器的参数列表放置在 jvalue 数组里，它们会全部传给Java构造器方法。
NewObjectV:
可将所有构造器参数放置于 va_list 里，它们也会在调用Java构造器方法时作为参数传入。
参数：
	•	env ：JNI接口指针
	•	clazz : 需要初始化对象的类
	•	methodID：构造器方法ID
	•	const jvalue *args: 构造器参数
	•	va_list args : 构造器函数
返回值：
返回一个Java对象，无法被构造时返回 NULL
抛出异常：
	•	InstantiationException 如果传入的 clazz 是一个抽象类或接口。
	•	OutOfMemoryError 如果系统没有内存时。
	•	其他所有构造器可能抛出的异常
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
    if (thr == NULL) stop("Unable to allocate debug thread's java/lang/Thread instance");
以上代码来至：OpenJDK源码中的 /hotspot/agent/src/share/native/jvmdi/sa.cpp

GetObjectClass
jclass GetObjectClass(JNIEnv *env, jobject obj);
获取一个对象的class。
参数：
	•	env ：JNI接口指针
	•	obj：需要获取class的对象( 必须 不能为 NULL )
返回值：
一个Java类对象（Java class object）
使用实例：
cls = env->GetObjectClass(self);
fid = env->GetFieldID(cls, "peer", "Ljava/awt/peer/ComponentPeer;");
env->SetObjectField(self, fid, lpeer);
以上代码来至: OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Frame.cpp

GetObjectRefType
jobjectRefType GetObjectRefType(JNIEnv* env, jobject obj);
返回对象的引用类型，参数obj可能是局部变量，全局变量，或弱全局变量。
参数：
	•	env ：JNI接口指针
	•	obj： 一个局部变量或全局或弱全局引用。
返回值：
如果一下的几种值（定义为 jobjectRefType 枚举值）
	•	无效引用： JNIInvalidRefType = 0
	•	局部引用： JNILocalRefType = 1
	•	全局引用： JNIGlobalRefType = 2
	•	弱全局引用：JNIWeakGlobalRefType = 3
无效引用是没有指向有效的值。指向的地址不是在内存中分配的地址。
例如 NULL 指向的都是无效引用。
GetObjectRefType 函数的 obj 参数不能是已经删除的引用。
使用实例：
jobjectRefType ref = env->GetObjectRefType(stringArray);
if(ref == JNIGlobalRefType) {
  env->DeleteGlobalRef(stringArray);
}
以上代码来至：https://github.com/wshackle/java4cpp/blob/7f705cd274ab0f42c3ca9cf28242b53c5b33b193/src/main/resources/cpp_easy_method_wrap.cpp

IsInstanceOf
jboolean IsInstanceOf(JNIEnv *env, jobject obj, jclass clazz);
检查一个对象是不是一个类的实例。
参数：
	•	env ：JNI接口指针
	•	obj：Java对象
	•	clazz：Java类对象（java class object）
返回值：
如果obj可以被cast成clazz，则返回 JNI_TURE ，否则返回 JNI_FALSE 。
注意 NULL 对象不能cast成任何clazz。
使用实例：
    jthrowable xcp = env->ExceptionOccurred();
    if (xcp != NULL) {
        env->ExceptionClear(); // if we don't do this, FindClass will fail

        jclass outofmem = env->FindClass("java/lang/OutOfMemoryError");
        DASSERT(outofmem != NULL);
        jboolean isOutofmem = env->IsInstanceOf(xcp, outofmem);

        env->DeleteLocalRef(outofmem);

        if (isOutofmem) {
            env->DeleteLocalRef(xcp);
            throw std::bad_alloc();
        } else {
            // rethrow exception
            env->Throw(xcp);
            return xcp;
        }
    }
以上代码来至：OpenJDK 源码 /jdk/src/windows/native/sun/windows/awt_new.cpp

IsSameObject
jboolean IsSameObject(JNIEnv *env, jobject ref1, jobject ref2);
检查两个引用是否指向同一个Java对象。
参数：
	•	env ：JNI接口指针
	•	ref1: java对象1
	•	ref2: java对象2
返回值：
如果指向的是同一个Java对象或两个都是 NULL，则返回 JNI_TRUE 。
否则返回 JNI_FALSE
使用实例：
// Trying to identify the same FontDescriptor by comparing the
// pointers.
refFontDescriptor = env->GetObjectArrayElement(array, i);
if (env->IsSameObject(refFontDescriptor, fontDescriptor)) {
  env->DeleteLocalRef(refFontDescriptor);
  env->DeleteLocalRef(array);
  return i;
}
env->DeleteLocalRef(refFontDescriptor);
以上代码来至：OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Font.cpp

4.8 访问对象成员域
GetFieldID
jfieldID GetFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
返回一个class的指定成员域的fieldID，成员域根据它的名字和签名来指定。返回的fieldID可以用来调用 Get<type>Field 和 Set<type>Field 来访问具体的成员域。
GetFieldID() 可能会导致一个没有被初始化（uninitialized）的类被初始化。
题外话：从JVM的源码来看，这个函数是首先回去尝试初始化这个类的.
GetFieldId() 不能用来获取数组的长度，使用 GetArrayLength() 来替代。
参数：
	•	env ：JNI接口指针
	•	clazz ：java class object
	•	name ：成员域的名称（MUTF-8编码）
	•	sig ：成员域的签名（MUTF-8编码）
返回值：
返回成员域的fieldID，操作失败则返回 NULL
抛出异常：
	•	NoSuchFieldError 如果没有指定的字段存在
	•	ExceptionInInitializerError 如果在初始化类的过程中出现异常
	•	OutOfMemoryError 如果系统内存不足时
使用实例：
cls = env->FindClass("java/awt/Button");
if (cls == NULL) {
  return;
}
AwtButton::labelID = env->GetFieldID(cls, "label", "Ljava/lang/String;");
DASSERT(AwtButton::labelID != NULL);
以上代码来至：OpenJDK 源码里的 /jdk/src/windows/native/sun/windows/awt_Button.cpp

Get<type>Field
jobject GetObjectField(JNIEnv *env, jobject obj, jfieldID fieldID);

jboolean GetBooleanField(JNIEnv *env, jobject obj, jfieldID fieldID);

jbyte GetByteField(JNIEnv *env, jobject obj, jfieldID fieldID);

jchar GetCharField(JNIEnv *env, jobject obj, jfieldID fieldID);

jshort GetShortField(JNIEnv *env, jobject obj, jfieldID fieldID);

jint GetIntField(JNIEnv *env, jobject obj, jfieldID fieldID);

jlong GetLongField(JNIEnv *env, jobject obj, jfieldID fieldID);

jfloat GetFloatField(JNIEnv *env, jobject obj, jfieldID fieldID);

jdouble GetDoubleField(JNIEnv *env, jobject obj, jfieldID fieldID);
这一系统Get方法，用于获取对象的非静态成员域。fieldID可以通过 GetFieldId() 函数来获取。
参数：
	•	env ：JNI接口指针
	•	object：java对象（必须不能为 NULL）
	•	fieldId ：一个有效的fieldID
返回值：
返回成员域的内容
使用实例：
bufferCapacityField = env->GetFieldID(bufferClass, "capacity", "I");
// ...
ret = env->GetIntField(buf, bufferCapacityField);
以上代码来至：OpenJDK 源码 /hotspot/src/share/vm/prims/jni.cpp

Set<type>Field
void SetObjectField(JNIEnv *env, jobject obj, jfieldID fieldID, jobject value);

void SetBooleanField(JNIEnv *env, jobject obj, jfieldID fieldID, jboolean value);

void SetByteField(JNIEnv *env, jobject obj, jfieldID fieldID, jbyte value);

void SetCharField(JNIEnv *env, jobject obj, jfieldID fieldID, jchar value);

void SetShortField(JNIEnv *env, jobject obj, jfieldID fieldID, jshort value);

void SetIntField(JNIEnv *env, jobject obj, jfieldID fieldID, jint value);

void SetLongField(JNIEnv *env, jobject obj, jfieldID fieldID, jlong value);

void SetFloatField(JNIEnv *env, jobject obj, jfieldID fieldID, jfloat value);

void SetDoubleField(JNIEnv *env, jobject obj, jfieldID fieldID, jdouble value);
一组设置基本类型成员域的函数。
参数：
	•	env ：JNI接口指针
	•	object：java对象（必须不能为 NULL）
	•	fieldId ：一个有效的fieldID
	•	value：设置的新值
返回值：
无
使用实例：
jint x = env->GetIntField(target, AwtComponent::xID);
jint y = env->GetIntField(target, AwtComponent::yID);
jint width = env->GetIntField(target, AwtComponent::widthID);
jint height = env->GetIntField(target, AwtComponent::heightID);

// ...
env->SetIntField(target, AwtComponent::widthID,  (jint) rc.right);
env->SetIntField(target, AwtComponent::heightID, (jint) rc.bottom);
以上代码来至：OpenJDK 源码中的 /jdk/src/windows/native/sun/windows/awt_Chooice.cpp


4.9 调用对象方法
GetMethodID
jmethodID GetMethodID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
返回一个类或接口的非静态实例方法MethodID，一个方法可以使 clazz 的方法，也可以使 clazz 从其父类继承而来的方法。这个方法靠方法名或方法签名来指定。
GetMethodID() 会触发一个没有初始化的类被初始化。
通过使用 <init> 作为方法名，void<V> 作为返回值来获取构造器的methodID。
参数：
	•	env ：JNI接口指针
	•	clazz：java class object
	•	name：方法名（MUTF-8编码）
	•	sig：方法签名（MUTF-8编码）
返回值：
返回方法的MethodID，如果找不到该方法则返回 NULL
抛出异常：
	•	NoSucMethodError ：如果指定的方法没有被找到
	•	ExceptionInInitializerError: 如果在初始化类的过程中出现异常
	•	OutOfMemoryError ； 如果系统内存不足时。
使用实例：
    // Start thread which receives commands from the SA.
    jclass threadClass = env->FindClass("java/lang/Thread");
    if (threadClass == NULL) stop("Unable to find class java/lang/Thread");
    jstring threadName = env->NewStringUTF("Serviceability Agent Command Thread");
    if (threadName == NULL) stop("Unable to allocate debug thread name");
    jmethodID ctor = env->GetMethodID(threadClass, "<init>", "(Ljava/lang/String;)V");
    if (ctor == NULL) stop("Unable to find appropriate constructor for java/lang/Thread");
以上代码来至：OpenJDK 源码 /hotspot/agent/src/share/native/jvmdi/sa.cpp

Call<type>Method(MethodA, MethodV)
void CallVoidMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
void CallVoidMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
void CallVoidMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jobject CallObjectMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jobject CallObjectMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jobject CallObjectMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jboolean CallBooleanMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jboolean CallBooleanMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jboolean CallBooleanMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jbyte CallByteMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jbyte CallByteMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jbyte CallByteMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jchar CallCharMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jchar CallCharMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jchar CallCharMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jshort CallShortMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jshort CallShortMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jshort CallShortMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jint CallIntMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jint CallIntMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jint CallIntMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jlong CallLongMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jlong CallLongMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jlong CallLongMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jfloat CallFloatMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jfloat CallFloatMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jfloat CallFloatMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);

jdouble CallDoubleMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...);
jdouble CallDoubleMethodA(JNIEnv *env, jobject obj, jmethodID methodID, const jvalue *args);
jdouble CallDoubleMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);
调用一个Java非静态实例方法，methodID 根据 GetMethodID() 来获取。
如果用这些函数来访问private方法或构造器方法，这MethodID必须来至真正的 obj 类，而不能是它们的父类。
Call<type>Method ：
可传入将所有需要传入给Java方法的参数放置在 methodID 参数之后，NewObject函数将接受三个参数，并将其传入到Java方法内。
Call<type>MethodA:
可将所有需要传入给Java方法的参数列表放置在 jvalue 数组里，它们会全部传给Java方法。
Call<type>MethodV:
可将Java方法的参数放置于 va_list 里，它们也会在调用Java方法时作为参数传入。
参数：
	•	env ：JNI接口指针
	•	obj : 调用方法的java object
	•	methodID：构造器方法ID
	•	const jvalue *args: 构造器参数
	•	va_list args : 构造器函数
返回值：
返回Java方法的返回结果。
抛出异常：
在执行Java方法时可能抛出的任何异常。
使用实例：
jobject target = w->GetTarget(env);
env->CallVoidMethod(target, AwtFrame::activateEmbeddingTopLevelMID);
env->DeleteLocalRef(target);
以上代码来至：OpenJDK 源码里面的 /jdk/src/windows/native/sun/windows/awt_Dialog.cpp

CallNonvirtual<type>Method(MethodA, MethodV)
NativeType CallNonvirtual<type>Method(JNIEnv *env, jobject obj, jclass clazz, jmethodID methodID, ...);

NativeType CallNonvirtual<type>MethodA(JNIEnv *env, jobject obj, jclass clazz, jmethodID methodID, const jvalue *args);

NativeType CallNonvirtual<type>MethodV(JNIEnv *env, jobject obj, jclass clazz, jmethodID methodID, va_list args);
这一系列的方法和上一个系列的方法类似，区别在于 Call<type>Method() 调用方法是基于 jobject ，而 CallNonvirtual<type>Method 调用方式是基于 jclass 对象，通常 jobject 传入子类对象， jclass 传入父类class，则可以调用其子类的父类方法，类似于java里面调用 super.method() 一样，即在子类里重写方法里可以调用其父类的方法。


作者：骆驼骑士
链接：https://www.jianshu.com/p/f5afb6d444f1
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


