第三章 JNI类型和数据结构
本章主要讨论JNI如何映射Java和C之间数据类型。
3.1 基本类型
Java类型
本地类型
描述
c类型
int
jint
signed 32 bits
根据平台不同
long
jlong
signed 64 bits
根据平台不同
byte
jbyte
signed 8 bits
根据平台不同




char
jchar
unsigned 16 bits
typedef unsigned short
short
jshort
singed 16 bits
typedef short
boolean
jboolean
unsigned 8 bits
typedef unsigned char
float
jfloat
32 bits
typedef float
double
jdouble
64 bits
typedef double
void
void
N/A
N/A
<u>以下为个人理解部分：</u>

从JVM的源码来看，JNI在不同平台上映射的类型是区别的，其中主要针对的是 jint jlong jbyte :
源码如下：
#ifdef TARGET_ARCH_x86
# include "jni_x86.h"
#endif
#ifdef TARGET_ARCH_sparc
# include "jni_sparc.h"
#endif
#ifdef TARGET_ARCH_zero
# include "jni_zero.h"
#endif
#ifdef TARGET_ARCH_arm
# include "jni_arm.h"
#endif
#ifdef TARGET_ARCH_ppc
# include "jni_ppc.h"
#endif
例如在x86架构下的定义：
  #define JNICALL
  typedef int jint;

#ifdef _LP64
  typedef long jlong;
#else
  typedef long long jlong;
#endif

#else
  #define JNIEXPORT __declspec(dllexport)
  #define JNIIMPORT __declspec(dllimport)
  #define JNICALL __stdcall

  typedef int jint;
  typedef __int64 jlong;
#endif

typedef signed char jbyte;

<u>以上为个人理解部分。</u>
为了方便还定义了true、false：
#define JNI_FALSE 0
#define JNI_TRUE  1
还定义了整数型 jsize 来表示大小：
typedef jint jsize;
3.2 引用类型
jni类型
Java类型
jobject
Object
|- jclass
java.lang.Class
|- jstring
java.lang.String
|- jarray
array
|----jobjectArray
Object[]
|----jbooleanArray
boolean[]
|----jbyteArray
byte[]
|----jcharArray
char[]
|----jshortArray
short[]
|----jintArray
int[]
|---- jlongArray
long[]
|---- jfloatArray
float[]
|---- jdoubleArray
double[]
|- jthrowable
java.lang.Throwable
在C的实现中，所有的JNI引用类型都定义和 jobject 一样的，例如：
typedef jobject jclass;
而在C++的实现中，所有的JNI引用类型都被定义为一个空的类，例如：
class _jobject {};
class _jclass : public _jobject {};
// ...
typedef _jobject *jobject;
typedef _jclass *jclass;
3.3 成员域和成员方法ID
成员域和成员方法都被定义为普通的C指针类型：
struct _jfieldID;              /* opaque structure */ 
typedef struct _jfieldID *jfieldID;   /* field IDs */ 
 
struct _jmethodID;              /* opaque structure */ 
typedef struct _jmethodID *jmethodID; /* method IDs */ 
3.4 jvalue类型
jvalue类型是一个c的union类型，定义如下：
typedef union jvalue {
    jboolean z; 
    jbyte    b; 
    jchar    c; 
    jshort   s; 
    jint     i; 
    jlong    j; 
    jfloat   f; 
    jdouble  d; 
    jobject  l; 
} jvalue;
3.5 类型签名
JNI使用Java虚拟机的签名类型描述符，如下：
类型签名
Java类型
Z
boolean
B
byte
C
char
S
short
I
int
J
long
F
float
D
double
L fully-qualified-class ;
fully-qualified-class
[type
type[]
(arg-types)ret-type
method type
例如有Java方法：
long f(int n, String s, int[] arr);
则类型签名为：
(ILjava/lang/String;[I)J
其中以此对应：
	•	I 对应 int n 参数
	•	Ljava/lang/String; 对应 String s 参数， 注意包名不用点，而用斜杠对应。结尾用一个分号。
	•	[I 对应 int[] arr 参数
	•	括号对应括号
	•	J 对应返回值 long 
3.6 MUTF-8字符串
JNI使用Modified UTF-8（MUTF-8）字符串来表示各种字符串类型。Java虚拟机里面也同样使用MUTF-8字符串。
MUTF-8和标准的UTF-8字符串是由区别的。具体的区别可以查看wiki说明。


作者：骆驼骑士
链接：https://www.jianshu.com/p/35848c03f2d5
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


