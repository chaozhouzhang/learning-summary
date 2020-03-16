# Java技术体系

|Java优势|
|----|
|结构严谨、面向对象的编程语言。|
|摆脱了硬件平台的束缚，一次编写，导出运行。|
|提供了一种相对安全的内存管理和访问机制，避免了绝大部分的内存泄露和指针越界的问题。|
|实现了热点代码检测和运行时编译及优化，使得Java应用能随着运行时间的增加而获得更高的性能。|
|有一套完善的应用程序接口，有来自商业机构和开源社区的第三方类库。|

|Java技术体系：功能|
|----|
|Java程序设计语言|
|各种硬件平台上的Java虚拟机|
|Class文件格式|
|Java API类库|
|来自商业机构和开源社区的第三方Java类库|

|JDK Java Development Kit 支持Java程序开发的最小环境|
|----|
|Java程序设计语言|
|Java虚拟机|
|Java API类库|

|JRE Java Runtime Environment 支持Java程序开发的标准环境|
|----|
|Java API类库中的Java SE API子集|
|Java虚拟机|


## Java技术体系所包含的内容

![Java技术体系所包含的内容](https://raw.githubusercontent.com/chaozhouzhang/learning-summary/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA-JVM%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E4%B8%8E%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5-%E7%AC%AC%E4%B8%89%E7%89%88/source/Java%E6%8A%80%E6%9C%AF%E4%BD%93%E7%B3%BB%E6%89%80%E5%8C%85%E5%90%AB%E7%9A%84%E5%86%85%E5%AE%B9.png)

|Java技术体系：服务领域|
|----|
|Java Card|
|Java ME Micro Edition|
|Java SE Standard Edition|
|Java EE Enterprise Edition|


|主流JVM|
|----|
|Sun HotSpot|
|BEA JRockit|
|IBM J9 VM|


|技术|解释|
|----|----|
|模块化|通过模块化实现按需部署、降低复杂性和维护成本。OSGi、Jigsaw等。|
|混合语言|基于Java虚拟机的语言。Clojure、JRuby、Groovy、Scala、JPython、Fantom等|
|多核并行|利用多个CPU核心提供的计算资源来协作完成复杂的计算任务。Fork/Join模式处理并行编程。|
|丰富语法|自动装箱、泛型、动态注解、枚举、可变长参数、遍历循环、二进制数的原生支持、switch语句支持字符串、<>操作符、异常处理的改进、简化变长参数方法调用、面向资源的try-catch-finally语句、Lambda表达式、函数式编程。|
|64位虚拟机|CPU支持64位架构，Java虚拟机支持64位系统。指针膨胀和各种数据类型对齐补白的内存问题。-XX:+UserCompressedOops普通对象指针压缩。|

|JDK|
|----|
|Apache Harmony|
|Sun JDK|
|Oracle JDK|
|Open JDK：GPL V2协议|

# 编译Open JDK

源码主页：http://openjdk.java.net/

源码下载主页：https://download.java.net/openjdk/jdk8

源码下载地址：http://www.java.net/download/openjdk/jdk8/promoted/b132/openjdk-8-src-b132-03_mar_2014.zip



