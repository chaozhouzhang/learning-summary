# 对象访问
```
在Java语言中，对象访问是如何进行的？
```


```
Object object = new Object();

Object object
Java栈的本地变量表中，reference类型数据。

new Object()
Java堆中存储了Object类型所有实例数据值的结构化内存。
根据具体类型以及虚拟机实现的对象内存布局的不同，这块内存的长度是不固定的。
Java堆中必须包含能查到对象类型数据的地址信息，这些类型数据包括对象类型、父类、实现的接口、方法等存储在方法区中。
```

|实例数据值 Instance Data|
|----|
|对象中各个实例字段的数据|


|对象内存布局 Object Memory Layout|
|----|
|对象头(Header)|
|实例数据(Instance Data)|
|对齐填充(Padding)|


|对象访问方式|
|----|
|句柄|
|直接指针|

## 1、句柄访问方式
```
Java堆划分出一块内存来做句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据和类型数据各自的具体地址信息。
```

|优势|
|----|
|reference中存储的是稳定的句柄地址。|

```
垃圾收集时移动对象是非常普遍的行为，在对象被移动的时候只会改变句柄中的实例数据指针，而reference本身不需要被修改。
```

![通过句柄访问对象](https://github.com/chaozhouzhang/learning-summary/blob/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA-JVM%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E4%B8%8E%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5-%E7%AC%AC%E4%B8%89%E7%89%88/source/%E9%80%9A%E8%BF%87%E5%8F%A5%E6%9F%84%E8%AE%BF%E9%97%AE%E5%AF%B9%E8%B1%A1.png)

## 2、直接指针访问方式

```
Java堆对象的布局中必须考虑如何放置访问类型数据的相关信息，reference中直接存储的就是对象地址。
```

|优势|
|----|
|访问速度更快，节省了一次指针定位的时间开销。|

|例子|
|----|
|Sun HotSpot虚拟机|

![通过直接指针访问对象](https://github.com/chaozhouzhang/learning-summary/blob/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA-JVM%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E4%B8%8E%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5-%E7%AC%AC%E4%B8%89%E7%89%88/source/%E9%80%9A%E8%BF%87%E7%9B%B4%E6%8E%A5%E6%8C%87%E9%92%88%E8%AE%BF%E9%97%AE%E5%AF%B9%E8%B1%A1.png)

