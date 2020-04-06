## Sun Javac编译器
javac编译器是一个由Java语言编写的程序，程序源码位于JDK的langtools中。
```
/langtools/src/share/classes/
```
线上地址：
```
https://hg.openjdk.java.net/jdk8/jdk8/langtools/
```


## Sun Javac的编译过程

1、解析与填充符号表过程
Parse and Enter

2、插入式注解处理器的注解处理过程
Annotation Processing

3、分析与字节码生成过程
Analyse Generate

Javac编译动作入口：
com.sun.tools.javac.main.JavaCompiler
compile()
compile2()

### 1、解析与填充符号表

#### 1.1、解析

parseFiles()

词法分析
将源代码的字符流转变为标记Token集合。
标记是编译过程的最小字符，关键字、变量名、字面量、运算符都可以称为标记。
词法分析由com.sun.tools.javac.parser.Scanner实现。

语法分析
根据Token序列来构造抽象语法树的过程。
抽象语法树 AST Abstract Syntax Tree 是一种用来描述程序代码语法结构的树形表达方式，语法树的每一个节点都代表着程序代码中的一个语法结构Contruct，例如包、类型、修饰符、运算符、接口、返回值、代码注释等都是一个语法结构。
语法分析由com.sun.tools.javac.parser.Parser实现，产生的语法树由com.sun.tools.javac.tree.JCTree表示。

#### 2、填充符号表

enterTrees()
符号表Symbol Table是由一组符号地址和符号信息构成的表格，符号表中所登记的信息在编译的不同阶段都要用到。

在语义分析中，符号表所登记的内容将用于语义检查和产生中间代码。
在目标代码生成阶段，当对符号名进行地址分配时，符号表是地址分配的依据。

填充符号表的过程由com.sun.tools.javac.comp.Enter实现，出口是待处理列表To Do List，包含每一个编译单元的抽象语法树的顶级节点，以及package-info.java。

#### 3、注解处理器
插入式注解处理器可以读取、修改、添加抽象语法树中的任何元素，并且编译器将回到Round解析以及填充符号表的过程重新处理。

插入式注解处理器的初始化过程在initProcessAnnotations()方法完成，执行过程在processAnnotations()方法完成。

#### 4、语义分析和字节码生成

##### 语义分析

1、标注检查
attribute()
变量使用前是否已经被声明、变量与赋值之间的数据类型是否能够匹配、常量折叠。
常量折叠使得程序运行期不用增加CPU指令的运算量。
标注检查由com.sun.tools.javac.comp.Attr和com.sun.tools.javac.comp.Check实现。

2、数据以及控制流分析
flow()
对程序上下文逻辑更进一步的验证。
可以检查出如程序局部变量在使用前是否有赋值、方法的每条路径是否都有返回值、是否所有的受查异常都被正确处理。
数据以及控制流分析由com.sun.tools.javac.comp.Flow实现。

局部变量与实例变量、类变量有区别，在常量池中没有CONSTANT_Fieldref_info的符号引用，没有访问标志Access_Flags的信息，将局部变量声明为final，对运行期没有影响，变量的不变性仅仅由编译器在编译期间保障。


