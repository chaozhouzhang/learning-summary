

LruCache 是一种内存缓存算法，全称为 Least Recently Used Cache，即近期最少使用算法。当我们需要对一些数据进行缓存以便下次进行展示时，我们就可以考虑使用该类进行相关数据的缓存。它比较普遍的使用场景是对从网络中下载下来的图片进行缓存，一来可以节省再次加载时的网络资源消耗，二来可以提高再次加载时的速度。

LruCache保存了对有限数量的值的强引用的缓存。每次访问一个值时，它都会被移动到队列的头部。当一个值被添加到一个完整的缓存中时，该队列末尾的值将被清除，并且可能有资格进行垃圾收集。
```
它是一个持有强引用类型的缓存。
每当访问缓存的值时，被访问值就会移动到缓存队列的头部。
当缓存队列存满时，它就会将队列尾部，也就是最不常被访问的值踢出去，以便保存新值。
```

LruCache 内部维护了一个 LinkedHashMap 的成员，通过对 LinkedHashMap 的操作，可以实现近期最少使用算法。


## 初始化

默认情况下，缓存大小是根据条目的数量来度量的。重写sizeOf()方法以在不同的单元中调整缓存的大小，返回每个缓存值的大小以便 LruCache 内部的缓存计算，不重写的话默认返回值为1。
例如，这个缓存仅限于4MiB的位图:
```
int cacheSize = 4 * 1024 * 1024; // 表示的是缓存的最大值 4MiB 

LruCache<String, Bitmap> bitmapCache = new LruCache<String, Bitmap>(cacheSize) {
		protected int sizeOf(String key, Bitmap value) {
     		return value.getByteCount();
     }
}};
```
## 读写缓存

key 值不能为空，并且在取得 Bitmap 的值之后应当进行非空判断才能使用，因为 get 方法有可能会返回 null。

```
bitmapCache.put("cache.png", bitmap);
```


```
Bitmap bitmap = bitmapCache.get("cache.png");
```
## create


如果缓存丢失，应根据相应键的需要计算，请重写create()方法。这简化了调用代码，允许它假设一个值总是会返回，即使在缓存失败的情况下也是如此。

这个方法默认的返回值是 null，它是在 get 方法内部被调用的，如果在 LruCache 内部没有 get 方法传入的 key 所对应的值时，就会通过回调 create 方法来创建该 key 的值 createdValue，需要注意的是如果此时该 key 值在 create 创建完该值之前恰好通过 put 方法缓存入了一个该 key 的新值，那么 createdValue 就会和这个新值发生冲突，这时候的决策是放弃 create 方法产生的值。



1、如果缓存的值包含需要显式释放的资源，则重写entryRemove()方法。


4、这个类是线程安全的。自动执行多个缓存操作，在缓存上同步:
```
synchronized (cache) {
    if (cache.get(key) == null) {
         cache.put(key, value);
    }
}
```

5、该类不允许将null用作键或值。get、put或remove的null返回值是明确的：也就是该键不在缓存中。


