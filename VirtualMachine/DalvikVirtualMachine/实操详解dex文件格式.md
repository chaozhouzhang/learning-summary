# 在Mac环境下配置dx终端指令
1、打开终端，进入HOME目录
```
cd $HOME
```
2、更新.bash_profile文件
```
touch .bash_profile
```
3、打开.bash_profile文件
```
open -e .bash_profile
```
4、加入dx文件所在路径，android sdk自带dx
```
export PATH=${PATH}:~/Library/Android/sdk/build-tools/29.0.3
```
5、保存并关闭.bash_profile文件
6、重启终端
7、验证配置结果
```
dx --version
```
8、结果
```
dx version 1.16
```



# 使用终端命令在Android中执行dex文件

0、创建Java源文件
```
public class HelloWorld {
    public static void main(String[] args) {
        print("Hello World!");
    }

    public static void print(String msg) {
        System.out.println(msg);
    }
}
```

1、编译Java源文件生成class文件
```
javac HelloWorld.java
```
2、编译class文件生成dex文件
```
dx --dex --output=HelloWorld.dex HelloWorld.class
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/VirtualMachine/DalvikVirtualMachine/HelloWorld-dex.png?raw=true)

3、进入手机系统
```
adb shell
```
创建dex存储目录
```
generic_x86:/ $ mkdir /data/local/tmp/dalvik-cache
generic_x86:/ $ export ANDROID_DATA=/data/local/tmp
generic_x86:/ $ exit 
```
4、将dex文件放进手机的dex存储目录
```
adb push HelloWorld.dex /data/local/tmp/dalvik-cache/
```
5、执行dex文件
```
adb shell dalvikvm -cp /data/local/tmp/dalvik-cache/HelloWorld.dex HelloWorld
```

6、输出结果
```
Hello World!
```


# dex文件格式概览




|名称|格式|说明|
|----|----|----|
|header|header_item|标头|
|string_ids|string_id_item[]|字符串标识符列表|
|type_ids|type_id_item[]|类型标识符列表|
|proto_ids|proto_id_item[]|方法原型标识符列表|
|field_ids|field_id_item[]|字段标识符列表|
|method_ids|method_id_item[]|方法标识符列表|
|class_defs|class_def_item[]|类定义列表|
|call_site_ids|call_site_id_item[]|调用点标识符列表|
|method_handles|method_handle_item[]|方法句柄列表|
|data|ubyte[]|数据区|
|link_data|ubyte[]|静态链接文件中使用的数据|




![](https://upload-images.jianshu.io/upload_images/1152636-4de9d36e8fdeb22d.jpg)

