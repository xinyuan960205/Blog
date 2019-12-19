# ConcurrentHashMap

## 一、结构

ConCurrentHashMap的**底层是：散列表+红黑树**，与HashMap是一样的。

### 1.1 JDK1.7

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/ConcurrentHashMapDS1.7.png)

在JDK1.7中，ConcurrentHashMap通过“锁分段”来实现线程安全。ConcurrentHashMap将哈希表分成许多片段（segments），每一个片段（table）都类似于HashMap，它有一个HashEntry数组，数组的每项又是HashEntry组成的链表。每个片段都是Segment类型的，Segment继承了ReentrantLock，所以Segment本质上是一个可重入的互斥锁。这样每个片段都有了一个锁，这就是“锁分段”。线程如想访问某一key-value键值对，需要先获取键值对所在的segment的锁，获取锁后，其他线程就不能访问此segment了，但可以访问其他的segment。

### 1.2 JDK1.8

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/ConcurrentHashMapDS1.8.png)

在JDK1.8中，ConcurrentHashMap放弃了“锁分段”，取而代之的是类似于HashMap的数组+链表+红黑树结构，使用CAS算法和synchronized实现线程安全。



顶部注释：

根据上面注释我们可以简单总结：

- JDK1.8底层是**散列表+红黑树**
- ConCurrentHashMap支持**高并发**的访问和更新，它是**线程安全**的
- 检索操作不用加锁，get方法是非阻塞的
- key和value都不允许为null