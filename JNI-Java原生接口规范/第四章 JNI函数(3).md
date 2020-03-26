第四章 JNI函数
4.10 访问静态域
GetStaticFieldID
jfieldID GetStaticFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
返回类的静态域的fieldID，这个静态域通过字段名和签名来指定。GetStatic<type>Field 和 SetStatic<type>Field 一系列访问函数通过该方法获得的fieldID来获取静态域。
此函数可能引起未初始化（initialized）的类初始化。
参数：
	•	env ：JNI接口指针
	•	clazz ：java class object
	•	name：静态域的名称
	•	sig ：静态域的签名
返回值：
返回fieldID，无法指定的静态域没有找到则返回 NULL
抛出异常：
	•	NoSuchFieldError ：如果指定的静态域没有被找到
	•	ExceptionInInitializerError：如果类在初始化的过程中出现异常。
使用实例：
  imagePath_ID = env->GetStaticFieldID(clazz, "imagePath", "Ljava/lang/String;");
  CHECK_EXCEPTION;

  symbolPath_ID = env->GetStaticFieldID(clazz, "symbolPath", "Ljava/lang/String;");
  CHECK_EXCEPTION;
以上代码来至：OpenJDK/hotspot/agent/src/os.win32/windbg/sawindbg.cpp

GetStatic<type>Field
jobject GetStaticObjectField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jboolean GetStaticBooleanField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jbyte GetStaticByteField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jchar GetStaticCharField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jshort GetStaticShortField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jint GetStaticIntField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jlong GetStaticLongField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jfloat GetStaticFloatField(JNIEnv *env, jclass clazz, jfieldID fieldID);
jdouble GetStaticDoubleField(JNIEnv *env, jclass clazz, jfieldID fieldID);
一系列函数用于获取不同类型的静态域。
参数：
	•	env ：JNI接口指针
	•	class：java class对象
	•	fieldId： 静态域的fieldId.
返回值：
返回静态域的值。
使用实例：
jfieldID fieldID = env->GetStaticFieldID(wfClass, fieldName, signature);
return env->GetStaticObjectField(wfClass, fieldID);
以上代码来至：OpenJDK/jdk/src/windows/native/sun/java2d/windows/WindowsFlags.cpp

SetStatic<type>Field
void SetStaticObjectField(JNIEnv *env, jclass clazz, jfieldID fieldID, jobject value);
void SetStaticBooleanField(JNIEnv *env, jclass clazz, jfieldID fieldID, jboolean value);
void SetStaticByteField(JNIEnv *env, jclass clazz, jfieldID fieldID, jbyte value);
void SetStaticCharField(JNIEnv *env, jclass clazz, jfieldID fieldID, jchar value);
void SetStaticShortField(JNIEnv *env, jclass clazz, jfieldID fieldID, jshort value);
void SetStaticIntField(JNIEnv *env, jclass clazz, jfieldID fieldID, jint value);
void SetStaticLongField(JNIEnv *env, jclass clazz, jfieldID fieldID, jlong value);
void SetStaticFloatField(JNIEnv *env, jclass clazz, jfieldID fieldID, jfloat value);
void SetStaticDoubleField(JNIEnv *env, jclass clazz, jfieldID fieldID, jdouble value);
一系列设置不同类型静态域的函数。
参数：
	•	env ：JNI接口指针
	•	class：java class对象
	•	fieldId： 静态域的fieldId.
	•	value：静态域的新值
返回值：
无
使用实例：
env->SetStaticBooleanField(wFlagsClassID, d3dEnabledID, d3dEnabled);
以上代码来至：OpenJDK/jdk/src/windows/native/sun/java2d/windows/WIndowsFlags.cpp


4.11 调用静态方法
GetStaticMethodID
jmethodID GetStaticMethodID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
返回类的静态方法的methodID，静态方法按方法名和签名来指定。
该方法可能引发未初始化的方法被初始化。
参数：
	•	env ：JNI接口指针
	•	class：java class对象。
	•	name：静态方法名（MUTF-8编码字符串）
	•	sig：静态方法签名（MUTF-8编码字符串）
返回值：
返回静态方法的methodID，或操作失败返回 NULL
抛出异常：
	•	NoSucMethodError ：如果指定的方法没有被找到
	•	ExceptionInInitializerError: 如果在初始化类的过程中出现异常
	•	OutOfMemoryError ； 如果系统内存不足时。
使用实例：
if (AwtCursor::updateCursorID == NULL) {
    jclass cls = env->FindClass("sun/awt/windows/WGlobalCursorManager");
    AwtCursor::globalCursorManagerClass =(jclass)env->NewGlobalRef(cls);
    AwtCursor::updateCursorID =env->GetStaticMethodID(cls, "nativeUpdateCursor", 
                                                      "(Ljava/awt/Component;)V");
    DASSERT(AwtCursor::globalCursorManagerClass != NULL);
    DASSERT(AwtCursor::updateCursorID != NULL);
}

env->CallStaticVoidMethod(AwtCursor::globalCursorManagerClass,
                          AwtCursor::updateCursorID, jcomp);
以上代码来至：OpenJDK/jdk/src/windows/native/sun/windows/awt_Cursor.cpp

CallStatic<type>Method(MethodA, MethodV)
NativeType CallStatic<type>Method(JNIEnv *env, jclass clazz, jmethodID methodID, ...);

NativeType CallStatic<type>MethodA(JNIEnv *env, jclass clazz, jmethodID methodID, jvalue *args);

NativeType CallStatic<type>MethodV(JNIEnv *env, jclass clazz, jmethodID methodID, va_list args);
一系列调用不同类型的静态方法的函数。
参数：
	•	env ：JNI接口指针
	•	clazz： Java class对象
	•	methodId: 静态方法methodID
返回值：
返回Java静态方法返回的结果。


4.12 操作字符串
NewString
jstring NewString(JNIEnv *env, const jchar *unicodeChars, jsize len);
使用unicode字符串数组来构造一个 java.lang.String 对象。
参数：
	•	env ：JNI接口指针
	•	unicodeChars ：指向unicode字符串的指针
	•	len : unicode字符串的长度
返回值：
返回一个Java String对象，当字符串无法被构造时返回 NULL
抛出异常：
	•	OutOfMemoryError 如果系统内存不足时。
使用实例：
jchar *strp = new jchar[len];
env->GetCharArrayRegion(str, off, len, strp);
jstring jstr = env->NewString(strp, len);
以上代码来至：OpenJDK/jdk/src/windows/native/sun/windows/awt_Font.cpp

GetStringLength
jsize GetStringLength(JNIEnv *env, jstring string);
返回Java String的长度（unicode字符串数组的长度）
参数：
	•	env ：JNI接口指针
	•	string：需要获取长度的字符串
返回值：
Java String的长度
使用实例：
int length = env->GetStringLength(title);
TCHAR *buffer = new TCHAR[length + 1];
env->GetStringRegion(title, 0, length, reinterpret_cast<jchar*>(buffer));
buffer[length] = L'\0';
VERIFY(::SetWindowText(w->GetHWnd(), buffer));
delete[] buffer;
以上代码来至：OpenJDK/jdk/src/windows/native/sun/windows/awt_Window.cpp

GetStringChars
const jchar * GetStringChars(JNIEnv *env, jstring string, jboolean *isCopy);
返回Java String对象的unicode字符串数组的指针。这个指针一直在调用 ReleaseStringchars() 方法前都有效。
参数：
	•	env ：JNI接口指针
	•	string：目标Java String对象
	•	isCopy ：NULL或JNI_TRUE表示返回一份copy，JNI_FALSE表示不copy，直接返回指向Java String的原始地址
返回值：
返回unicode 字符串的指针。操作失败而返回 NULL
使用实例：
// Set the certificate's friendly name
int size = env->GetStringLength(jCertAliasName);
pszCertAliasName = new WCHAR[size + 1];

jCertAliasChars = env->GetStringChars(jCertAliasName, NULL);
memcpy(pszCertAliasName, jCertAliasChars, size * sizeof(WCHAR));
pszCertAliasName[size] = 0; // append the string terminator
以上代码来至：OpenJDK/jdk/src/windows/native/sun/securty/mscapi/security.cpp
个人理解部分：
	1.	默认 isCopy 参数传 NULL 即可以了，默认返回拷贝值，但如果要求返回Java String对象的原始地址，则一定不要修改它的值，否则破坏了Java String不可修改的原则。
	2.	先要记住jchar的定义 typedef unsigned short jchar; 
	3.	然后了解wchat_t 宽字节， 在C语言里面的定义 typedef unsigned short wchar_t; 在C++里面则是内置类型。
	4.	然后要明白 unicode(UTF-16) 和 UTF-8 的区别。UTF-16永远是双字节，而UTF-8在英文是单字节，中文等才用双字节。
	5.	在Java里面的字符串都是使用 unicode 双字节的。GetStringChars方法返回的都是双字节，类似C语言里面的 wchat_t 类型。
	6.	UTF-8字符串以’\0’结尾，而Unicode字符串不是。

ReleaseStringChars
void ReleaseStringChars(JNIEnv *env, jstring string, const jchar *chars);
通知虚拟机本地代码不再需要访问 chars ，chars参数是一个指针，指向用 string 参数调用 GetStringChars() 方法返回的unicode字符串。
参数：
	•	env ：JNI接口指针
	•	string：一个java字符串对象
	•	chars：指向unicode字符串的指针
返回值：
无
使用实例：
// jstring jCertAliasName

// Set the certificate's friendly name
int size = env->GetStringLength(jCertAliasName);
pszCertAliasName = new WCHAR[size + 1];

jCertAliasChars = env->GetStringChars(jCertAliasName, NULL);
memcpy(pszCertAliasName, jCertAliasChars, size * sizeof(WCHAR));
pszCertAliasName[size] = 0; // append the string terminator

CRYPT_DATA_BLOB friendlyName = {
  sizeof(WCHAR) * (size + 1),
  (BYTE *) pszCertAliasName
};

env->ReleaseStringChars(jCertAliasName, jCertAliasChars);
以上代码来至：OpenJDK/jdk/src/windows/native/sun/securty/mscapi/security.cpp

NewStringUTF
jstring NewStringUTF(JNIEnv *env, const char *bytes);
根据一个MUTF-8编码的字符串数组(an array of characters in modified UTF-8 encodin)来构建一个 java.lang.String对象。
参数：
	•	env ：JNI接口指针
	•	bytes：指向MUTF-8编码字符串的指针。
返回值：
返回一个Java String对象，或失败时返回 NULL
抛出异常：
	•	OutOfMemoryError 如果系统内存不足时。
使用实例：
jstring threadName = env->NewStringUTF("Serviceability Agent Command Thread");
以上代码来至：OpenJDK/hotspot/agent/src/share/native/jvmdi/sa.cpp

GetStringUTFLength
jsize GetStringUTFLength(JNIEnv *env, jstring string);
返回代表字符串的MUTF-8编码字符串数组长度(the length in bytes of the modified UTF-8 representation of a string)
参数：
	•	env ：JNI接口指针
	•	string：Java String对象
返回值：
返回字符串的MUTF-8编码字符串数组的长度。
使用实例：
static char* jstr_to_utf(JNIEnv *env, jstring str, TRAPS) {

  char* utfstr = NULL;

  if (str == NULL) {
    THROW_0(vmSymbols::java_lang_NullPointerException());
    //throw_new(env,"NullPointerException");
  }

  int len = env->GetStringUTFLength(str);
  int unicode_len = env->GetStringLength(str);

  utfstr = NEW_RESOURCE_ARRAY(char, len + 1);

  env->GetStringUTFRegion(str, 0, unicode_len, utfstr);

  return utfstr;
}
以上代码来至：OpenJDK/hotspot/src/share/vm/prims/pref.cpp

GetStringUTFChars
const char * GetStringUTFChars(JNIEnv *env, jstring string, jboolean *isCopy);
返回一个指针，指向代表字符串MUTF-8编码的字节数组（an array of bytes representin the string in modified UTF-8 encoding）。这个数组的指针一个在调用 ReleaseStrinUTFChars()前有效。
参数：
	•	env ：JNI接口指针
	•	string：目标Java String对象
	•	isCopy ：NULL或JNI_TRUE表示返回一份copy，JNI_FALSE表示不copy，直接返回指向Java String的原始地址
返回值：
返回指向MUTF-8字符串的指针，或失败返回 NULL
使用实例：
const char* prop  = env->GetStringUTFChars(pProp, JNI_FALSE);
const char* value = env->GetStringUTFChars(pValue, JNI_FALSE);
jboolean   retval = uPtr->set_option(prop, value);
env->ReleaseStringUTFChars(pProp,  prop);
env->ReleaseStringUTFChars(pValue, value);
以上代码来至：OpenJDK/jdk/src/share/native/com/sun/java/util/jar/pack/jni.cpp

ReleaseStringUTFChars
void ReleaseStringUTFChars(JNIEnv *env, jstring string, const char *utf);
通知Java虚拟机本地代码不需要访问 utf 了。这个 utf 是使用 string 调用 GetStringUTFChars 返回的指针。
参数：
	•	env ：JNI接口指针
	•	string：Java String对象
	•	utf：指向MUTF-8字符串的指针
返回值：
无
使用实例：
    jstring value = (jstring)env->CallObjectMethod(type, windowTypeNameMID);
    if (value == NULL) {
        env->DeleteLocalRef(type);
        return;
    }

    const char* valueNative = env->GetStringUTFChars(value, 0);
    if (valueNative == NULL) {
        env->DeleteLocalRef(value);
        env->DeleteLocalRef(type);
        return;
    }

    // ...

    env->ReleaseStringUTFChars(value, valueNative);
    env->DeleteLocalRef(value);
    env->DeleteLocalRef(type);
以上代码来至：OpenJDK/jdk/src/windows/native/sun/windows/awt_Window.cpp

GetStringRegion
void GetStringRegion(JNIEnv *env, jstring str, jsize start, jsize len, jchar *buf);
从 start 偏移量开始拷贝len 个unicode字符到指定的 buf 缓存中。
参数：
	•	env ：JNI接口指针
	•	str： 目标Java String对象
	•	start : 偏移量
	•	len：拷贝的长度
	•	buf : 拷贝到的目标缓存buffer
返回值：
无
抛出异常：
	•	StringIndexOutOfBoundsException 当指针越界时
使用实例：
size_t length = env->GetStringLength(javaWarningString) + 1;
warningString = new WCHAR[length];
env->GetStringRegion(javaWarningString, 0,
                     static_cast<jsize>(length - 1), reinterpret_cast<jchar*>(warningString));
warningString[length-1] = L'\0';
以上代码来至：OpenJDK/jdk/src/windows/native/sun/windows/awt_Window.cpp
个人理解部分：
GetStringRegion函数不需要VM来分配内存，而直接将Java字符串拷贝到C++字符串数组中，因此不需要释放函数，也不会造成 OutOfMemoryError

GetStringUTFRegion
void GetStringUTFRegion(JNIEnv *env, jstring str, jsize start, jsize len, char *buf);
从 start 偏移量开始拷贝len 个unicode字符，将其转换成MUTF-8编码，放置到指定的 buf 缓存中。
参数：
	•	env ：JNI接口指针
	•	str： 目标Java String对象
	•	start : 偏移量
	•	len：拷贝的长度
	•	buf : 拷贝到的目标缓存buffer
返回值：
无
抛出异常：
	•	StringIndexOutOfBoundsException 当指针越界时
使用实例：
static char* jstr_to_utf(JNIEnv *env, jstring str, TRAPS) {

  char* utfstr = NULL;

  if (str == NULL) {
    THROW_0(vmSymbols::java_lang_NullPointerException());
    //throw_new(env,"NullPointerException");
  }

  int len = env->GetStringUTFLength(str);
  int unicode_len = env->GetStringLength(str);

  utfstr = NEW_RESOURCE_ARRAY(char, len + 1);

  env->GetStringUTFRegion(str, 0, unicode_len, utfstr);

  return utfstr;
}
以上代码来至：OpenJDK/hotspot/src/share/vm/prims/pref.cpp
个人理解部分：
GetStringUTFRegion函数不需要VM来分配内存，而直接将Java字符串拷贝到C++字符串数组中，因此不需要释放函数，也不会造成 OutOfMemoryError

GetStringCritical
const jchar * GetStringCritical(JNIEnv *env, jstring string, jboolean *isCopy);
和GetStringChars函数类似，如果可能，Java虚拟机会返回内部指向字符串元素的指针，否则返回一个复制值。但不管怎样，GetStringCritical 和 ReleaseStringCritical 这些函数的使用必须非常小心，在这两个函数之间，本地代码必须不能调用任何可能阻塞当前线程的JNI函数。
参数：
	•	env ：JNI接口指针
	•	jstring : 目标Java字符串对象
	•	isCopy ：是否复制值。
返回值：
虚拟机内部指向字符串元素的指针。
使用实例：
JNIEXPORT jstring JNICALL Java_com_study_jnilearn_Sample_sayHello
  (JNIEnv *env, jclass cls, jstring j_str)
{
    const jchar* c_str= NULL;
    char buff[128] = hello ;
    char* pBuff = buff + 6;
    /*
     * 在GetStringCritical/RealeaseStringCritical之间是一个关键区。
     * 在这关键区之中,绝对不能呼叫JNI的其他函数和会造成当前线程中断或是会让当前线程等待的任何本地代码，
     * 否则将造成关键区代码执行区间垃圾回收器停止运作，任何触发垃圾回收器的线程也会暂停。
     * 其他触发垃圾回收器的线程不能前进直到当前线程结束而激活垃圾回收器。
     */
    c_str = (*env)->GetStringCritical(env,j_str,NULL);   // 返回源字符串指针的可能性
    if (c_str == NULL)  // 验证是否因为字符串拷贝内存溢出而返回NULL
    {
        return NULL;
    }
    while(*c_str) 
    {
        *pBuff++ = *c_str++;
    }
    (*env)->ReleaseStringCritical(env,j_str,c_str);
    return (*env)->NewStringUTF(env,buff);
}
以上代码来至：http://www.2cto.com/kf/201412/364156.html
以下为个人理解部分：
因为仅从文档说明，这两个函数较为难以理解，所以这里摘抄一段网上的说明文字：
Get/ReleaseStringChars和Get/ReleaseStringUTFChars这对函数返回的源字符串会后分配内存，如果有一个字符串内容相当大，有1M左右，而且只需要读取里面的内容打印出来，用这两对函数就有些不太合适了。此时用Get/ReleaseStringCritical可直接返回源字符串的指针应该是一个比较合适的方式。不过这对函数有一个很大的限制，在这两个函数之间的本地代码不能调用任何会让线程阻塞或等待JVM中其它线程的本地函数或JNI函数。因为通过GetStringCritical得到的是一个指向JVM内部字符串的直接指针，获取这个直接指针后会导致暂停GC线程，当GC被暂停后，如果其它线程触发GC继续运行的话，都会导致阻塞调用者。所以在Get/ReleaseStringCritical这对函数中间的任何本地代码都不可以执行导致阻塞的调用或为新对象在JVM中分配内存，否则，JVM有可能死锁。另外一定要记住检查是否因为内存溢出而导致它的返回值为NULL，因为JVM在执行GetStringCritical这个函数时，仍有发生数据复制的可能性，尤其是当JVM内部存储的数组不连续时，为了返回一个指向连续内存空间的指针，JVM必须复制所有数据。
来源：http://www.2cto.com/kf/201412/364156.html

ReleaseStringCritical
void ReleaseStringCritical(JNIEnv *env, jstring string, const jchar *carray);
释放由 GetStringCritical 函数调用获得的字符串指针。
参数：
	•	env ：JNI接口指针
	•	string：字符串对象
	•	carray: 获取到的字符串指针。
返回值：
无

4.13 操作数组
GetArrayLength
jsize GetArrayLength(JNIEnv *env, jarray array);
返回数组(array)的元素个数。
参数：
	•	env ：JNI接口指针
	•	array：目标Java数组对象
返回值：
返回数组的长度
使用实例：
    int length = env->GetArrayLength(andMask);
    jbyte *andMaskPtr = new jbyte[length];
    env->GetByteArrayRegion(andMask, 0, length, andMaskPtr);
以上代码来至：awt_Cursor.cpp (openjdk\jdk\src\windows\native\sun\windows)

NewObjectArray
jobjectArray NewObjectArray(JNIEnv *env, jsize length, jclass elementClass, jobject initialElement);
构造一个新的 elementClass 类型的数组，并设置其初始值。
参数：
	•	env ：JNI接口指针
	•	length：数组长度
	•	elementClass ：数组元素的class
	•	initialElement：初始化数组元素
返回值：
返回一个Java数组对象，如果数组不能被构造则返回 NULL。
抛出异常：
	•	OutOfMemoryError ：如果系统内存不足时。
使用实例：
clauseReading = env->NewObjectArray(cClause, JNU_ClassString(env), NULL);
for (int i  =0; i < cClause; i++) {
  env->SetObjectArrayElement(clauseReading, i, rgClauseReading[i]);
}
以上代码来至：awt_Component.cpp (openjdk\jdk\src\windows\native\sun\windows)

GetObjectArrayElement
jobject GetObjectArrayElement(JNIEnv *env, jobjectArray array, jsize index);
返回数组的index处的元素对象。
参数：
	•	env ：JNI接口指针
	•	array：Java数组
	•	index：数组下标
返回值：
返回一个Java对象。
抛出异常：
	•	ArrayIndexOutOfBoundsException： 如果index数组越界时。
使用实例：
        jsize i;
        int itemCount = env->GetArrayLength(items);
        if (itemCount > 0) {
           c->SendMessage(WM_SETREDRAW, (WPARAM)FALSE, 0);
           for (i = 0; i < itemCount; i++)
           {
               jstring item = (jstring)env->GetObjectArrayElement(items, i);
               JNI_CHECK_NULL_GOTO(item, "null item", next_elem);
               c->SendMessage(CB_INSERTSTRING, index + i, JavaStringBuffer(env, item));
               env->DeleteLocalRef(item);
next_elem:
               ;
           }
           c->SendMessage(WM_SETREDRAW, (WPARAM)TRUE, 0);
           InvalidateRect(c->GetHWnd(), NULL, TRUE);
           c->ResetDropDownHeight();
           c->VerifyState();
        }
以上代码来至：awt_Choice.cpp (openjdk\jdk\src\windows\native\sun\windows)

SetObjectArrayElement
void SetObjectArrayElement(JNIEnv *env, jobjectArray array, jsize index, jobject value);
设置数组的某个值。
参数：
	•	env ：JNI接口指针
	•	array：一个Java数组
	•	index：数组下标
	•	value：设置的新值
返回值：
无
抛出异常:*
	•	ArrayIndexOutOfBoundsException ，如果 index 数组越界时。
	•	ArrayStoreException ，如果 value 类型错误时。
使用实例：
clauseReading = env->NewObjectArray(cClause, JNU_ClassString(env), NULL);
for (int i  =0; i < cClause; i++) {
  env->SetObjectArrayElement(clauseReading, i, rgClauseReading[i]);
}
以上代码来至：awt_Component.cpp (openjdk\jdk\src\windows\native\sun\windows)

New<PrimitiveType>Array
jbooleanArray NewBooleanArray(JNIEnv *env, jsize length);
jbyteArray NewByteArray(JNIEnv *env, jsize length);
jcharArray NewCharArray(JNIEnv *env, jsize length);
jshortArray NewShortArray(JNIEnv *env, jsize length);
jintArray NewIntArray(JNIEnv *env, jsize length);
jlongArray NewLongArray(JNIEnv *env, jsize length);
jfloatArray NewFloatArray(JNIEnv *env, jsize length);
jdoubleArray NewDoubleArray(JNIEnv *env, jsize length);
一组函数用于构造各种不同基本类型的数组对象。
参数：
	•	env ：JNI接口指针
	•	length：数组长度
返回值：
返回Java数组，数组无法被构造则返回 NULL。

Get<PrimitiveType>ArrayElements
jboolean GetBooleanArrayElements(JNIEnv *env, jbooleanArray array, jboolean *isCopy);
jbyte GetByteArrayElements(JNIEnv *env, jbyteArray array, jboolean *isCopy);
jchar GetCharArrayElements(JNIEnv *env, jcharArray array, jboolean *isCopy);
jshort GetShortArrayElements(JNIEnv *env, jshortArray array, jboolean *isCopy);
jint GetIntArrayElements(JNIEnv *env, jintArray array, jboolean *isCopy);
jlong GetLongArrayElements(JNIEnv *env, jlongArray array, jboolean *isCopy);
jfloat GetFloatArrayElements(JNIEnv *env, jfloatArray array, jboolean *isCopy);
jdouble GetDoubleArrayElements(JNIEnv *env, jdoubleArray array, jboolean *isCopy);
一组函数用于获取各种不同基本类型数组的元素。
返回的结构在调用相应的 Release<Type>ArrayElements() 之前一直有效。因为返回的可能是一个复制值，因此对于返回值的修改不一定会对原始数组产生影响，直到调用了相应的 Release<Type>ArrayElements() 才可能影响到原始数组。
参数：
	•	env ：JNI接口指针
	•	array：目标数组
	•	isCopy: NULL或 JNI_FALSE 不COPY，JNI_TRUE 返回COPY值。
返回值：
返回数组某个元素的指针。或失败返回 NULL 。

Release<PrimitiveType>ArrayElements
void ReleaseBooleanArrayElements(JNIEnv *env, jbooleanArray array, jboolean *elems, jint mode);
void ReleaseByteArrayElements(JNIEnv *env, jbyteArray array, jbyte *elems, jint mode);
void ReleaseCharArrayElements(JNIEnv *env, jcharArray array, jchar *elems, jint mode);
void ReleaseShortArrayElements(JNIEnv *env, jshortArray array, jshort *elems, jint mode);
void ReleaseIntArrayElements(JNIEnv *env, jintArray array, jint *elems, jint mode);
void ReleaseLongArrayElements(JNIEnv *env, jlongArray array, jlong *elems, jint mode);
void ReleaseFloatArrayElements(JNIEnv *env, jfloatArray array, jfloat *elems, jint mode);
void ReleaseDoubleArrayElements(JNIEnv *env, jdoubleArray array, jdouble *elems, jint mode);
一组函数通知虚拟机本地代码不再需要访问数组中的元素。这些元素是使用 Get<type>ArrayElements函数来获取的，如果需要，这个函数会把修改后的内容复制到原始的数组中。使用最后一个参数 mode 来控制：
	•	mode = 0 , 将内容复制回原始数组，并释放 elems （通常情况下都传入0即可）
	•	mode = JNI_COMMIT ，将内容复制回原始数组，但不释放 elems 
	•	mode = JNI_ABORT ，不将内容复制回原始数组，并释放 elems 
参数：
	•	env ：JNI接口指针
返回值：
无

Get<PrimitiveType>ArrayRegion
void GetBooleanArrayRegion(JNIEnv *env, jbooleanArray array, jsize start, jsize len,  jboolean *buf);
void GetByteArrayRegion(JNIEnv *env, jbyteArray array, jsize start, jsize len,  jbyte *buf);
void GetCharArrayRegion(JNIEnv *env, jcharArray array, jsize start, jsize len,  jchar *buf);
void GetShortArrayRegion(JNIEnv *env, jshortArray array, jsize start, jsize len,  jshort *buf);
void GetIntArrayRegion(JNIEnv *env, jintArray array, jsize start, jsize len,  jint *buf);
void GetLongArrayRegion(JNIEnv *env, jlongArray array, jsize start, jsize len,  jlong  *buf);
void GetFloatArrayRegion(JNIEnv *env, jfloatArray array, jsize start, jsize len,  jfloat *buf);
void GetDoubleArrayRegion(JNIEnv *env, jdoubleArray array, jsize start, jsize len,  jdouble *buf);
一组函数用来复制基本类型数据的一部分值到buffer。
参数：
	•	env ：JNI接口指针
	•	array：java数组
	•	start ：起始的下标
	•	len：需要复制的元素个数
	•	buf：目标buffer
返回值：
无
抛出异常：
	•	ArrayIndexOutOfBoundsException ：如果数组越界时。
使用实例：
    (*env)->GetBooleanArrayRegion(env, jArray, 0, *ckpLength, jpTemp);
    if ((*env)->ExceptionCheck(env)) {
        free(jpTemp);
        return;
    }
以上代码来至：

Set<PrimitiveType>ArrayRegion
void SetBooleanArrayRegion(JNIEnv *env, jbooleanArray array, jsize start, jsize len, const jboolean *buf);
void SetByteArrayRegion(JNIEnv *env, jbyteArray array, jsize start, jsize len, const jbyte *buf);
void SetCharArrayRegion(JNIEnv *env, jcharArray array, jsize start, jsize len, const jchar *buf);
void SetShortArrayRegion(JNIEnv *env, jshortArray array, jsize start, jsize len, const jshort *buf);
void SetIntArrayRegion(JNIEnv *env, jintArray array, jsize start, jsize len, const jint *buf);
void SetLongArrayRegion(JNIEnv *env, jlongArray array, jsize start, jsize len, const jlong  *buf);
void SetFloatArrayRegion(JNIEnv *env, jfloatArray array, jsize start, jsize len, const jfloat *buf);
void SetDoubleArrayRegion(JNIEnv *env, jdoubleArray array, jsize start, jsize len, const jdouble *buf);
一组从buffer复制回基本类型数组的函数。
参数：
	•	env ：JNI接口指针
	•	array：java数组
	•	start ：起始的下标
	•	len：需要复制的元素个数
	•	buf：数据源buffer
返回值：
无
抛出异常：
	•	ArrayIndexOutOfBoundsException ：如果数组越界时。

GetPrimitiveArrayCritical
void * GetPrimitiveArrayCritical(JNIEnv *env, jarray array, jboolean *isCopy);
和 Get<type>ArrayElements 函数类似，但如果可能的话，Java虚拟机会返回一个基本类型数组的原始指针，否则返回一个copy值得指针。
GetPrimitiveArrayCritical 和 ReleasePrimitiveArrayCritical 这些函数的使用必须非常小心，在这两个函数之间，本地代码必须不能调用任何可能阻塞当前线程以等待其他Java线程的JNI函数，或任何系统调用。（例如，当前线程不能 read一个另一个Java线程正在写入的流。）
参数：
	•	env ：JNI接口指针
	•	array：目标数组
	•	isCopy：JNI_TRUE复制值，JNI_FALSE不复制。
返回值：
无

ReleasePrimitiveArrayCritical
void ReleasePrimitiveArrayCritical(JNIEnv *env, jarray array, void *carray, jint mode);
释放由 GetPrimitiveArrayCritical函数获取的数组原始指针。
参数：
	•	env ：JNI接口指针
	•	array：目标数组
	•	carray: 之前获取的原始数组指针
	•	mode：0 或 JNI_COMMIT 或 JNI_ABORT 
返回值：
无
使用实例：
  jint len = (*env)->GetArrayLength(env, arr1);
  jbyte *a1 = (*env)->GetPrimitiveArrayCritical(env, arr1, 0);
  jbyte *a2 = (*env)->GetPrimitiveArrayCritical(env, arr2, 0);
  /* We need to check in case the VM tried to make a copy. */
  if (a1 == NULL || a2 == NULL) {
    ... /* out of memory exception thrown */
  }
  memcpy(a1, a2, len);
  (*env)->ReleasePrimitiveArrayCritical(env, arr2, a2, 0);
  (*env)->ReleasePrimitiveArrayCritical(env, arr1, a1, 0);


4.14 注册本地方法
RegisterNatives
jint RegisterNatives(JNIEnv *env, jclass clazz, const JNINativeMethod *methods, jint nMethods);
给clazz注册一个本地方法。
JNINativeMethod定义为：
typedef struct {
  char *name;
  char *signature;
  void *fnPtr;
} JNINativeMethod;
这些本地函数必须至少有2个参数，且前2个参数必须依次是：JNIEnv *env ，jobject objectOrClass
例如：
jstring nativeFunc(JNIEnv* env, jobject thiz)
{
    return env->NewStrinUTF("Hello world from native method!");    
}
参数：
	•	env ：JNI接口指针
	•	clazz：指定的java class对象
	•	methods：原生方法列表
	•	nMethods： 原生方法列表数量
返回值：
成功返回0， 失败返回负数
抛出异常：
	•	NoSuchMethodError： 如果指定的方法没有找到，或者方法不是native。
使用实例：
jstring native_stringFromJNI( JNIEnv* env, jobject thiz )
{
    return (*env)->NewStringUTF(env, "Hello from JNI !");
}
static const char *classPathName = "com/example/hellojni/HelloJni";

static JNINativeMethod methods[] = {
        {"stringFromJNI", "()Ljava/lang/String;", (void*)native_stringFromJNI},
};

jint JNI_OnLoad(JavaVM* vm, void* reserved)
{
    //注册本地方法.Load 目标类
    clazz = (*env)->FindClass(env,classPathName);
    if (clazz == NULL)
    {
        LOGE("Native registration unable to find class '%s'", classPathName);
        return JNI_ERR;
    }
    //注册本地native方法
    if((*env)->RegisterNatives(env, clazz, methods, NELEM(methods)) < 0)
    {
        LOGE("ERROR: MediaPlayer native registration failed\n");
           return JNI_ERR;
    }
    return JNI_VERSION_1_6; 
}
以上代码来至：http://blog.chinaunix.net/uid-26009923-id-3410141.html

UnregisterNatives
jint UnregisterNatives(JNIEnv *env, jclass clazz);
注销一个类的所有本地方法，这个类会回到之前被链接或注册本地方法之前的状态。
这个函数不应该在一般的本地代码中使用。
参数：
	•	env ：JNI接口指针
	•	clazz：java class对象
返回值：
成功返回0， 失败返回负数。
使用实例：
jclass jClass = env->FindClass(JNI_AN_MainActivity);
env->UnregisterNatives(jClass);
env->DeleteLocalRef(jClass);
以上代码来至：https://github.com/Hackerofshi/DemoJNI-master/blob/01d974076bb157f6257a81ee52e70db4b96e8802/app/src/main/jni/jni_lib.cpp



作者：骆驼骑士
链接：https://www.jianshu.com/p/85ab777e14b2
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


