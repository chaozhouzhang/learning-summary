### 1、HashMap数据结构概览

数组+单链表+红黑树

数组默认大小是多少？

为什么数组的默认大小是16？

数组大小的最大值是多少？

为什么数组大小的最大值是这么多？

数组在什么时候进行扩容？

为什么在数组的0.75的时候进行扩容？

数组在扩容的时候扩大多少？

为什么扩大这么多？

HashMap的数组长度要取2的整次幂？



什么是扰动函数？扰动函数怎么加大哈希/散列值低位的随机性？


数组插入节点的时候怎么判断要插入到哪个位置？为什么要这样插入？

什么是哈希/散列值HashCode？什么是哈希算法？什么是哈希函数？为什么要优化？怎么优化？为什么要使用散列值优化函数也就是扰动函数对哈函数得到的初始哈希/散列值进行优化？

# 概览

|HashMap包含的数据结构|
|----|
|数组：是有序的元素序列|
|单链表：是一种链式存取的数据结构，用一组地址任意的存储单元存放线性表中的数据元素|
|红黑树：是一种自平衡二叉查找树|

|位/比特/bit/b|
|----|
|数据存储的最小单位，在计算机中的二进制数系统中，每个0或1就是一个位。|
|计算机中的CPU位数指的是CPU一次能处理的最大位数。|

|位操作|解释|
|----|----|
|~|NOT 按位取反|
|`|`|OR 按位或|
|^|XOR 按位异或|
|&|AND 按位与|
|<<|左移运算符|
|>>|右移运算符|
|>>>|无符号右移|

```
class Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```
```
class TreeNode<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
}
```

# HashMap的键值对插入过程
```
HashMap hashMap = new HashMap();
hashMap.put("android","Android技术堆栈");
```
## 1、如果数组是空的或者数组的长度为0

如果数组是空的或者数组的长度为0，则对数组进行调整：
```
tab = resize()
```

使用`数组容量的初始化默认值`作为数组大小，来初始化数组
使用`加载因子的默认值`与`数组容量的初始化默认值`的乘积作为`阈值`，来决定何时调整数组
```
// zero initial threshold signifies using defaults
newCap = DEFAULT_INITIAL_CAPACITY;
newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/HashMap-%E5%88%9D%E5%A7%8B%E5%8C%96%E6%95%B0%E7%BB%84.png?raw=true)


|数组容量|
|----|
|数组容量就是数组大小|
|必须是2的幂|

|数组容量的初始化默认值|
|----|
|1 << 4 也就是2的4次幂 16|


|数组容量的最大值|
|----|
|1 << 30 也就是2的30次幂 |


|阈值|
|----|
|阈值是负载系数与数组容量的乘积|
|如果数组中已存储的节点数量达到阈值，则需要进行数组调整，以存储更多的数据|


|负载系数/满载率/载荷因子/负荷系数/加载因子|
|----|
|负载系数就是阈值与数组容量的比值|
|如果数组中已存储的节点数量达到阈值，则需要进行数组调整，以存储更多的数据|

|负载系数的默认值|
|----|
|0.75f|



## 2、计算出节点即将存储在数组中的位置
如果key为空，直接返回0，也就是key为空的节点将直接存储在数组的0位置；
如果key不为空，则`key的初始散列值`与`key的初始散列值的16位右移`进行按位异或，得到优化之后的散列值。
```
//散列值优化函数，也称扰动函数
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
右位移16位，正好是32bit的一半，自己的高半区和低半区按位异或，就是为了混合原始哈希码的高位和低位，以此来加大低位的随机性。而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来。
Java 8只做一次扰动，而Java 7做了4次，多了可能边际效用也不大，效率也会降低。

初始散列值

```
public native int hashCode();
```
```
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
但是问题是，这样就算散列值分布再松散，要是只取最后几位的话，碰撞也会很严重。更要命的是如果散列本身做得不好，分布上成等差数列的漏洞，恰好使最后几个低位呈现规律性重复，就无比蛋疼。
所以这个时候我们就要使用扰动函数，对哈希函数得到的初始哈希/散列值进行优化，得到一个新的具有更大低位随机性的哈希/散列值。

将数组大小-1与优化之后的key的散列值进行按位与
```
(n - 1) & hash
```

对初始哈希/散列值做低位掩码，取数组下标。因为这样数组长度-1正好相当于一个低位掩码，进行与操作的结果就是散列值的高位全部归零，只保留低位值，用来做数组下标访问。相当于取模，也就是哈希/散列值%(16-1)。
什么是低位掩码？
```
//散列值与数组长度16-1的与操作：

	10100101 11000100 00100101
&	00000000 00000000 00001111
----------------------------------
	00000000 00000000 00000101    //高位全部归零，只保留末四位
```

```
public class HashTest {
    public static void main(String[] args) {
        int h = "android".hashCode();
        System.out.println(h);
        String h2 = Integer.toBinaryString(h);
        System.out.println(h2);

        int move = h >>> 16;
        System.out.println(move);
        String move2 = Integer.toBinaryString(move);
        System.out.println(move2);

        int hash = h ^ move;
        System.out.println(hash);
        String hash2 = Integer.toBinaryString(hash);
        System.out.println(hash2);
    }
}
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/HashMap-%E8%AE%A1%E7%AE%97%E5%AD%98%E5%82%A8%E4%BD%8D%E7%BD%AE.png?raw=true)
## 2、如果计算出的位置所对应的存储为空，则将节点插入此位置中

```
tab[i] = newNode(hash, key, value, null);
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/HashMap-%E5%AD%98%E5%82%A8%E5%88%B0%E6%95%B0%E7%BB%84.png?raw=true)
## 3、如果计算出的数组位置所对应的存储不为空

### 3.1、如果之前已经存储相同key的节点
如果onlyIfAbsent为false，或者之前所存储的value值为空，则覆盖之前所存储节点的value值
```
e = p;
```
```
// existing mapping for key
V oldValue = e.value;
if (!onlyIfAbsent || oldValue == null)
    e.value = value;
```
![](https://github.com/chaozhouzhang/learning-summary/blob/master/HashMap-%E8%A6%86%E7%9B%96%E5%B7%B2%E5%AD%98%E5%82%A8%E8%8A%82%E7%82%B9%E7%9A%84%E5%80%BC.png?raw=true)

### 3.2、如果计算出的数组位置所对应的存储节点为树节点
```
e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```

