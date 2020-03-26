# Android NDK概览


Android NDK 原生开发套件，是一个工具集，开发者可以使用 C 和 C++ 等语言以原生代码实现应用的各个部分。

```
class MyActivity : Activity() {
  /**
   * Native method implemented in C/C++
   */
  external fun computeFoo()
}
```
|NDK作用|
|----|
|进一步提升设备性能，以降低延迟，或运行计算密集型应用，如游戏或物理模拟。|
|重复使用自己或其他开发者的现有的 C 或 C++ 库。|
|在平台之间移植其应用。|

|Android Studio NDK 开发步骤|
|----|
|使用NDK，将C和C++代码编译到原生库中。|
|使用CMake或ndk-build搭配Gradle，编译原生库。|
|使用集成编译系统Gradle，将原生库打包到APK中。|
|使用Java代码，通过Java原生接口JNI框架，调用原生库中的函数。|
|使用LLDB，调试原生代码。|


|Android Studio编译原生库的编译工具|
|----|
|CMake 默认编译工具|
|ndk-build |


|Android Studio创建原生项目步骤|
|----|
|1、创建支持C/C++的新项目。|
|2、创建新的原生源文件。|
|3、创建CMake编译脚本CMakeLists.txt，指示CMake如何将原生源文件编译入库。|
|3.1、或创建ndk-build编译脚本Android.mk，指示ndk-build如何将原生源文件编译入库。
|4、提供指向CMake或ndk-build脚本文件的路径，将Gradle关联到原生库。|
|5、编译并运行应用，Gradle使用编译脚本将源代码导入Android Studio项目，以依赖项的形式添加 CMake 或 ndk-build 进程，用于编译原生库，并将原生库打包到Apk中。|


|Android编译原生应用时使用的主要组件|解释|
|----|----|
|原生共享库|NDK从C/C++源代码编译原生共享库/.so文件|
|原生静态库|NDK从C/C++源代码编译原生静态库/.a文件，可将静态库关联到其他库。|
|Java原生接口JNI|JNI是Java和C++组件用以互相通信的接口。见Java原生接口规范。|
|应用二进制接口ABI|ABI精确定义应用的机器代码在运行时应该如何与系统交互。见ABI管理。|
|清单|如果应用不包含Java组件，则必须在清单中声明NativeActivity类。|

其中，NDK根据ABI定义编译.so文件，不同的ABI对应不同的架构。目前NDK只为32位ARM、AArch64、x86及x86-64提供ABI支持。

|使用 NDK 编译代码的方法|
|----|
|基于 Make 的 ndk-build。|
|CMake。|
|独立工具链，用于与其他编译系统集成，或与基于 configure 的项目搭配使用。|



## CMake实战
### 1、配置CMakeLists.txt
```
add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             native-lib.cpp)
find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )
              
target_link_libraries( # Specifies the target library.
                       native-lib
                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

### 2、配置Gradle
```
android {
    defaultConfig {
        externalNativeBuild {
            cmake {
                cppFlags "-frtti -fexceptions"
            }
        }
        ndk {
            // Specifies the ABI configurations of your native
            // libraries Gradle should build and package with your APK.
            abiFilters   'x86','x86_64', 'armeabi-v7a', 'arm64-v8a'
        }
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }
    }
}

```
### 3、编写原生代码
#### 3.1、方式一，静态方式，直接根据IDE提示生成JNI方法
```C
extern "C" JNIEXPORT jstring JNICALL
Java_android_stack_ndk_MainActivity_stringFromJNI(
        JNIEnv *env,
        jobject /* this */) {
    std::string hello = "Hello NDK.";
    return env->NewStringUTF(hello.c_str());
}
```
#### 3.2、方式二，动态方式，使用JNI_OnLoad注册本地方法

```C
JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved) {
	registerNativeMethods(env, JNI_REG_CLASS,
                             method_table, NELEM(method_table));
	return JNI_VERSION_1_4;
}
```

## 案例
使用NDK进行AES加解密，并进行apk的签名校验。GitHub地址：

https://github.com/chaozhouzhang/AndroidNDK

