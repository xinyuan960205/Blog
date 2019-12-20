# ConcurrentHashMap

## 一、结构

ConCurrentHashMap的**底层是：散列表+红黑树**，与HashMap是一样的。

### 1.1 JDK1.7

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/ConcurrentHashMapDS1.7.png)

在JDK1.7中，ConcurrentHashMap通过“锁分段”来实现线程安全。ConcurrentHashMap将哈希表分成许多片段（segments），每一个片段（table）都类似于HashMap，它有一个HashEntry数组，数组的每项又是HashEntry组成的链表。每个片段都是Segment类型的，Segment继承了ReentrantLock，所以Segment本质上是一个可重入的互斥锁。这样每个片段都有了一个锁，这就是“锁分段”。线程如想访问某一key-value键值对，需要先获取键值对所在的segment的锁，获取锁后，其他线程就不能访问此segment了，但可以访问其他的segment。

### 1.2 JDK1.8

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/ConcurrentHashMapDS1.8.png)

在JDK1.8中，ConcurrentHashMap放弃了“锁分段”，取而代之的是类似于HashMap的数组+链表+红黑树结构，使用CAS算法和synchronized实现线程安全。

## 二、源码分析

### 2.1 顶部注释

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/concurrenthashmap%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.webp)

根据上面注释我们可以简单总结：

- JDK1.8底层是**散列表+红黑树**
- ConCurrentHashMap支持**高并发**的访问和更新，它是**线程安全**的
- 检索操作不用加锁，get方法是非阻塞的
- key和value都不允许为null

### 2.2 内部类

- Node。最基本的内部类，key-value键值对，不支持setValue方法。
- TreeNode。红黑树节点，供TreeBins使用。
- TreeBin。红黑树结构。该类并不包装key-value键值对，而是TreeNode的列表和它们的根节点。这个类含有读写锁。
- ForwardingNode。不是传统的节点，不包含key-value键值对，包含一个nextTable指针，和find方法 。

### 2.3 域

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/concurrenthashmap%E5%9F%9F.webp)

### 2.4 put方法

#### 2.4.1 putVal(K key, V value, boolean onlyIfAbsent) 

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //计算哈希值
    int hash = spread(key.hashCode());
    int binCount = 0;
    //死循环，只有插入成功才能跳出循环
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        //如果table没有初始化，初始化table
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //根据哈希值计算在table中的位置
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            //如果这个位置没有值，直接将键值对放进去，不需要加锁
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //如果要插入的位置是一个forwordingNode节点，表示正在扩容，那么当前线程帮助扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //进行到这一步，说明要插入的位置有值，需要加锁
            synchronized (f) {
                //确定f是tab中的头节点
                if (tabAt(tab, i) == f) {
                    //如果头结点的哈希值大于等于0，说明要插入的节点在链表中
                    if (fh >= 0) {
                        binCount = 1;
                        //遍历链表中的所有节点
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果某一节点的key哈希值和key与参数相等，替换节点的value
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            //遍历到了最后一个节点，还没找到key对应的节点，根据参数新建节点，插入链表尾部
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //如果要插入的节点在树中，则按照树的方式插入或替换节点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //如果binCount不为0，说明插入或者替换操作完成了
            if (binCount != 0) {
                //判断节点数量是否大于8，如果大于就需要把链表转化成红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;//如果在链表中找到了指定key的节点，返回被替换的value
                break;
            }
        }
    }
    //能执行到这一步，说明节点不是被替换的，是被插入的，所以要将map的元素数量加1
    addCount(1L, binCount);
    return null;
}
```

可以将步骤总结如下：

1. 计算key哈希值
2. 根据哈希值计算在table中的位置
3. 根据哈希值执行插入或替换操作
   1. 如果这个位置没有值，直接将键值对放进去，不需要加锁。
   2. 如果要插入的位置是一个forwordingNode节点，表示正在扩容，那么当前线程帮助扩容
   3. 加锁。以下操作都需要加锁。
   4. 如果要插入的节点在链表中，遍历链表中的所有节点，如果某一节点的key哈希值和key与参数相等，替换节点的value，记录被替换的值；如果遍历到了最后一个节点，还没找到key对应的节点，根据参数新建节点，插入链表尾部。
   5. 如果要插入的节点在树中，则按照树的方式插入或替换节点。如果是替换操作，记录被替换的值
4. 判断节点数量是否大于8，如果大于就需要把链表转化成红黑树
5. 如果操作3中执行的是替换操作，返回被替换的value。程序结束。
6. 能执行到这一步，说明节点不是被替换的，是被插入的，所以要将map的元素数量加1。

#### 2.4.2 initTable()

接下来，我们来看看初始化散列表的时候干了什么事：`initTable()`

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/initTable.webp)

**只让一个线程对散列表进行初始化**！

#### 2.4.3 addCount(long x, int check)

可以看出，修改table结构使用了synchronized。进入addCount方法看看，

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```

可以看出，修改table大小时使用了CAS算法。

### 2.5 get方法

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    //计算key的哈希值
    int h = spread(key.hashCode());
    //如果表不为空，表长度大于0，key所在的桶的头结点不为null
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {//如果查到的桶的头结点的key哈希值与参数key的哈希值相同
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))//如果查到的桶的头结点的key参数key相等，返回桶的头结点的value
                return e.val;
        }
        else if (eh < 0)//如果查到的桶的头结点的key哈希值小于0，即要找的在树上
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {//如果要找的节点既不是桶的头结点也不是在树上，那就说明在链表上
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))//遍历链表，找到节点，返回值
                return e.val;
        }
    }
    //如果都没找到，返回null
    return null;
}
```

可以将步骤总结如下：

1. 通过key计算哈希值
2. 通过哈希值找到桶
3. 通过哈希值和桶来查找节点
   1. 以此判断桶的头结点是不是要找的节点
   2. 如果不是，判断桶的头节点的哈希值是否小于0，如果是则说明要找的节点在树上
   3. 如果以上两个条件都不满足，则说明要找的节点在链表上，遍历链表，查找节点

4. 如果通过以上步骤找到了节点，返回节点的value。没找到，就返回null。

从顶部注释我们可以读到，get方法是**不用加锁**的，是非阻塞的。

我们可以发现，Node节点是重写的，设置了volatile关键字修饰，致使它每次获取的都是**最新**设置的值

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%86%85%E7%BD%AEnode%E6%BA%90%E7%A0%81.webp)

## 三、总结

上面简单介绍了ConcurrentHashMap的核心知识，还有很多知识点都没有提及到，作者的水平也不能将其弄懂~~有兴趣进入的同学可到下面的链接继续学习。

下面我来简单总结一下ConcurrentHashMap的核心要点：

- **底层结构是散列表(数组+链表)+红黑树**，这一点和HashMap是一样的。
- Hashtable是将所有的方法进行同步，效率低下。而ConcurrentHashMap作为一个高并发的容器，它是通过**部分锁定+CAS算法来进行实现线程安全的**。CAS算法也可以认为是**乐观锁**的一种~
- 在高并发环境下，统计数据(计算size…等等)其实是无意义的，因为在下一时刻size值就变化了。
- get方法是非阻塞，无锁的。重写Node类，通过volatile修饰next来实现每次获取都是**最新**设置的值
- **ConcurrentHashMap的key和Value都不能为null**

## 参考

- <https://blog.csdn.net/panweiwei1994/article/details/78897275>
- <https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484161&idx=1&sn=6f52fb1f714f3ffd2f96a5ee4ebab146&chksm=ebd74200dca0cb16288db11f566cb53cafc580e08fe1c570e0200058e78676f527c014ffef41&scene=21###wechat_redirect>





