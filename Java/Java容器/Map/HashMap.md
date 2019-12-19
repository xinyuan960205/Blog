# HashMap

## 一、HashMap结构

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/HashMapDateStructure.jpg)

从上图中可以很清楚的看到，HashMap的数据结构是数组+链表+红黑树（红黑树since JDK1.8）。我们常把数组中的每一个节点称为一个桶。当向桶中添加一个键值对时，首先计算键值对中key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这种现象称为碰撞，这时按照尾插法(jdk1.7及以前为头插法)的方式添加key-value到同一hash值的元素的后面，链表就这样形成了。当链表长度超过8(TREEIFY_THRESHOLD)时，链表就转换为红黑树。

## 二、源码解析

### 2.1 顶部注释

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/HashMap%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.png)

翻译原文：

> HashMap是Map接口基于哈希表的实现。这种实现提供了所有可选的Map操作，并允许key和value为null（除了HashMap是unsynchronized的和允许使用null外，HashMap和HashTable大致相同。）。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。
> 此实现假设哈希函数在桶内适当地分布元素，为基本实现(get 和 put)提供了稳定的性能。迭代 collection 视图所需的时间与 HashMap 实例的“容量”（桶的数量）及其大小（键-值映射关系数）成比例。如果遍历操作很重要，就不要把初始化容量initial capacity设置得太高（或将加载因子load factor设置得太低），否则会严重降低遍历的效率。
> HashMap有两个影响性能的重要参数：初始化容量initial capacity、加载因子load factor。容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。initial capacity*load factor就是当前允许的最大元素数目，超过initial capacity*load factor之后，HashMap就会进行rehashed操作来进行扩容，扩容后的的容量为之前的两倍。
> 通常，默认加载因子 (0.75) 在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少rehash操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生rehash 操作。
> 如果很多映射关系要存储在 HashMap 实例中，则相对于按需执行自动的 rehash 操作以增大表的容量来说，使用足够大的初始容量创建它将使得映射关系能更有效地存储。
> 注意，此实现不是同步的。如果多个线程同时访问一个哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须保持外部同步。（结构上的修改是指添加或删除一个或多个映射关系的任何操作；仅改变与实例已经包含的键关联的值不是结构上的修改。）这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的非同步访问，如下所示： 
> Map m = Collections.synchronizedMap(new HashMap(…));
> 由所有此类的“collection 视图方法”所返回的迭代器都是fail-fast 的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的remove方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒在将来不确定的时间发生任意不确定行为的风险。
> 注意，迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测bug。

从上面的内容中可以总结出以下几点：

- 底层：HashMap是Map接口基于哈希表的实现。
- 是否允许null：HashMap允许key和value为null。
- 是否有序：HashMap不保证映射的顺序，特别是它不保证该顺序恒久不变。
- 何时rehash：超出当前允许的最大容量。initial capacity*load factor就是当前允许的最大元素数目，超过initial capacity*load factor之后，HashMap就会进行rehashed操作来进行扩容，扩容后的的容量为之前的两倍。
- 初始化容量对性能的影响：不应设置地太小，设置地小虽然可以节省空间，但会频繁地进行rehash操作。rehash会影响性能。总结：小了会增大时间开销（频繁rehash）；大了会增大空间开销（占用了更多空间）和时间开销（影响遍历）。
- 加载因子对性能的影响：加载因子过高虽然减少了空间开销，但同时也增加了查询成本。0.75是个折中的选择。总结：小了会增大时间开销（频繁rehash）；大了会也增大时间开销（影响遍历）。
- 是否同步：HashMap不是同步的。
- 迭代器：迭代器是fast-fail的。

### 2.2 定义

先来看看HashMap的定义：

public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
从中我们可以了解到：

HashMap<K,V>：HashMap是以key-value形式存储数据的。
extends AbstractMap<K,V>：继承了AbstractMap，大大减少了实现Map接口时需要的工作量。
implements Map<K,V>：实现了Map，提供了所有可选的Map操作。
implements Cloneable：表明其可以调用clone()方法来返回实例的field-for-field拷贝。
implements Serializable：表明该类是可以序列化的。

- 类继承图

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/hashmap%E7%B1%BB%E7%BB%A7%E6%89%BF%E5%9B%BE.png)

### 2.3 属性

#### 2.3.1 静态全局变量

```java
/**
 * 默认初始化容量，值为16
 * 必须是2的n次幂.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 最大容量, 容量不能超出这个值。如果一个更大的初始化容量在构造函数中被指定，将被MAXIMUM_CAPACITY替换.
 * 必须是2的倍数。最大容量为1<<30，即2的30次方。
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认的加载因子。
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 将链表转化为红黑树的临界值。
 * 当添加一个元素被添加到有至少TREEIFY_THRESHOLD个节点的桶中，桶中链表将被转化为树形结构。
 * 临界值最小为8
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 恢复成链式结构的桶大小临界值
 * 小于TREEIFY_THRESHOLD，临界值最大为6
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 桶可能被转化为树形结构的最小容量。当哈希表的大小超过这个阈值，才会把链式结构转化成树型结构，否则仅采取扩容来尝试减少冲突。
 * 应该至少4*TREEIFY_THRESHOLD来避免扩容和树形结构化之间的冲突。
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/hashmap%E9%9D%99%E6%80%81%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F.jpg)

#### 2.3.2 内部类Node

```java
/**
 * HashMap的节点类型。既是HashMap底层数组的组成元素，又是每个单向链表的组成元素
 */
static class Node<K,V> implements Map.Entry<K,V> {
    //key的哈希值
    final int hash;
    final K key;
    V value;
    //指向下个节点的引用
    Node<K,V> next;
    //构造函数
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/hashmap%E5%86%85%E9%83%A8%E7%B1%BB.jpg)

## 三、核心方法

### 3.1 构造函数

#### HashMap( int initialCapacity, float loadFactor)

```java
/**
 * 使用指定的初始化容量initial capacity 和加载因子load factor构造一个空HashMap
 *
 * @param  initialCapacity 初始化容量
 * @param  loadFactor      加载因子
 * @throws IllegalArgumentException 如果指定的初始化容量为负数或者加载因子为非正数。
 */
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/hashmap%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0.jpg)

在上面的构造方法最后一行，我们会发现调用了`tableSizeFor()`

```java
   /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

```java
    public static int numberOfLeadingZeros(int i) {
        // HD, Count leading 0's
        if (i <= 0)
            return i == 0 ? 32 : 0;
        int n = 31;
        if (i >= 1 << 16) { n -= 16; i >>>= 16; }
        if (i >= 1 <<  8) { n -=  8; i >>>=  8; }
        if (i >= 1 <<  4) { n -=  4; i >>>=  4; }
        if (i >= 1 <<  2) { n -=  2; i >>>=  2; }
        return n - (i >>> 1);
    }
```

这是位运算算法

看完上面可能会感到奇怪的是：**为啥是将2的整数幂的数赋给threshold**？

- threshold这个成员变量是阈值，决定了是否要将散列表再散列。它的值应该是：`capacity * loadfactor`才对的。

其实这里仅仅是一个初始化，当创建哈希表的时候，它会重新赋值的：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/%E5%88%9B%E5%BB%BAhash%E8%A1%A8%E9%87%8D%E6%96%B0%E8%B5%8B%E5%80%BC.jpg)

#### HashMap( int initialCapacity)

```java
/**
 * 使用指定的初始化容量initial capacity和默认加载因子DEFAULT_LOAD_FACTOR（0.75）构造一个空HashMap
 *
 * @param  initialCapacity 初始化容量
 * @throws IllegalArgumentException 如果指定的初始化容量为负数
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

#### HashMap()

```java
/**
 * 使用指定的初始化容量（16）和默认加载因子DEFAULT_LOAD_FACTOR（0.75）构造一个空HashMap
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

#### **HashMap( Map<**? extends K, ? extends V>m)

```java
/**
 * 使用指定Map m构造新的HashMap。使用指定的初始化容量（16）和默认加载因子DEFAULT_LOAD_FACTOR（0.75）
 * @param   m 指定的map
 * @throws  NullPointerException 如果指定的map是null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

### 3.2 哈希

#### hash( Object key)

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。计算位置的方法如下

```java
(n - 1) & hash1
```

其中的n为数组的长度，hash为hash(key)计算得到的值。

```java
/**
 * 计算key的哈希值。
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

从代码中可以看到，计算位置分为三步，第一步，取key的hashCode，第二步，key的hashCode高16位异或低16位，第三步，将第一步和第二部得到的结果进行取模运算。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/hashFunc.jpg)

看到这里有个疑问，为什么要做异或运算？

设想一下，如果n很小，假设为16的话，那么n-1即为15（0000 0000 0000 0000 0000 0000 0000 1111），这样的值如果跟hashCode()直接做与操作，实际上只使用了哈希值的后4位。如果当哈希值的高位变化很大，低位变化很小，这样很容易造成碰撞，所以把高低位都参与到计算中，从而解决了这个问题，而且也不会有太大的开销。

### 3.2 put方法

#### put( K key, V value)

```java
/**
 * 将指定参数key和指定参数value插入map中，如果key已经存在，那就替换key对应的value
 * 
 * @param key 指定key
 * @param value 指定value
 * @return 如果value被替换，则返回旧的value，否则返回null。当然，可能key对应的value就是null。
 */
public V put(K key, V value) {
    //putVal方法的实现就在下面
    return putVal(hash(key), key, value, false, true);
}
```

从源码中可以看到，put(K key, V value)可以分为三个步骤：

1. 通过hash(Object key)方法计算key的哈希值。
2. 通过putVal(hash(key), key, value, false, true)方法实现功能。
3. 返回putVal方法返回的结果。

计算key的哈希值已经在前面讲过。

#### putVal( int hash, K key, V value, boolean onlyIfAbsent,boolean evict)

```java
/**
 * Map.put和其他相关方法的实现需要的方法
 * 
 * @param hash 指定参数key的哈希值
 * @param key 指定参数key
 * @param value 指定参数value
 * @param onlyIfAbsent 如果为true，即使指定参数key在map中已经存在，也不会替换value
 * @param evict 如果为false，数组table在创建模式中
 * @return 如果value被替换，则返回旧的value，否则返回null。当然，可能key对应的value就是null。
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果哈希表为空，调用resize()创建一个哈希表，并用变量n记录哈希表长度
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //如果指定参数hash在表中没有对应的桶，即为没有碰撞
    if ((p = tab[i = (n - 1) & hash]) == null)
        //直接将键值对插入到map中即可
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //如果碰撞了，且桶中的第一个节点就匹配了
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //将桶中的第一个节点记录起来
            e = p;
        //如果桶中的第一个节点没有匹配上，且桶内为红黑树结构，则调用红黑树对应的方法插入键值对
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //不是红黑树结构，那么就肯定是链式结构
        else {
            //遍历链式结构
            for (int binCount = 0; ; ++binCount) {
                //如果到了链表尾部
                if ((e = p.next) == null) {
                    //在链表尾部插入键值对
                    p.next = newNode(hash, key, value, null);
                    //如果链的长度大于TREEIFY_THRESHOLD这个临界值，则把链变为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    //跳出循环
                    break;
                }
                //如果找到了重复的key，判断链表中结点的key值与插入的元素的key值是否相等，如果相等，跳出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                //用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        //如果key映射的节点不为null
        if (e != null) { // existing mapping for key
            //记录节点的vlaue
            V oldValue = e.value;
            //如果onlyIfAbsent为false，或者oldValue为null
            if (!onlyIfAbsent || oldValue == null)
                //替换value
                e.value = value;
            //访问后回调
            afterNodeAccess(e);
            //返回节点的旧值
            return oldValue;
        }
    }
    //结构型修改次数+1
    ++modCount;
    //判断是否需要扩容
    if (++size > threshold)
        resize();
    //插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

putVal方法可以分为下面的几个步骤：

1. 如果哈希表为空，调用resize()创建一个哈希表。
2. 如果指定参数hash在表中没有对应的桶，即为没有碰撞，直接将键值对插入到哈希表中即可。
3. 如果有碰撞，遍历桶，找到key映射的节点 
   1. 桶中的第一个节点就匹配了，将桶中的第一个节点记录起来。
   2. 如果桶中的第一个节点没有匹配，且桶中结构为红黑树，则调用红黑树对应的方法插入键值对。
   3. 如果不是红黑树，那么就肯定是链表。遍历链表，如果找到了key映射的节点，就记录这个节点，退出循环。如果没有找到，在链表尾部插入节点。插入后，如果链的长度大于TREEIFY_THRESHOLD这个临界值，则使用treeifyBin方法把链表转为红黑树。
4. 如果找到了key映射的节点，且节点不为null 
   1. 记录节点的vlaue。
   2. 如果参数onlyIfAbsent为false，或者oldValue为null，替换value，否则不替换。
   3. 返回记录下来的节点的value。
5. 如果没有找到key映射的节点（2、3步中讲了，这种情况会插入到hashMap中），插入节点后size会加1，这时要检查size是否大于临界值threshold，如果大于会使用resize方法进行扩容。

#### resize()

向hashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，hashMap就需要扩大数组的长度，以便能装入更多的元素。当然数组是无法自动扩容的，扩容方法使用一个新的数组代替已有的容量小的数组。

resize方法非常巧妙，因为每次扩容都是翻倍，与原来计算（n-1）&hash的结果相比，节点要么就在原来的位置，要么就被分配到“原位置+旧容量”这个位置。

```java
/**
 * 对table进行初始化或者扩容。
 * 如果table为null，则对table进行初始化
 * 如果对table扩容，因为每次扩容都是翻倍，与原来计算（n-1）&hash的结果相比，节点要么就在原来的位置，要么就被分配到“原位置+旧容量”这个位置。
 */
final Node<K,V>[] resize() {
    //新建oldTab数组保存扩容前的数组table
    Node<K,V>[] oldTab = table;
    //使用变量oldCap扩容前table的容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //保存扩容前的临界值
    int oldThr = threshold;
    int newCap, newThr = 0;
    //如果扩容前的容量 > 0
    if (oldCap > 0) {
        //如果当前容量>=MAXIMUM_CAPACITY
        if (oldCap >= MAXIMUM_CAPACITY) {
            //扩容临界值提高到正无穷
            threshold = Integer.MAX_VALUE;
            //无法进行扩容，返回原来的数组
            return oldTab;
        }
        //如果现在容量的两倍小于MAXIMUM_CAPACITY且现在的容量大于DEFAULT_INITIAL_CAPACITY
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&oldCap >= DEFAULT_INITIAL_CAPACITY)
            //临界值变为原来的2倍
            newThr = oldThr << 1; 
    }//如果旧容量 <= 0，而且旧临界值 > 0
    else if (oldThr > 0) 
        //数组的新容量设置为老数组扩容的临界值
        newCap = oldThr;
    else {//如果旧容量 <= 0，且旧临界值 <= 0，新容量扩充为默认初始化容量，新临界值为DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {//在当上面的条件判断中，只有oldThr > 0成立时，newThr == 0
        //ft为临时临界值，下面会确定这个临界值是否合法，如果合法，那就是真正的临界值
        float ft = (float)newCap * loadFactor;
        //当新容量< MAXIMUM_CAPACITY且ft < (float)MAXIMUM_CAPACITY，新的临界值为ft，否则为Integer.MAX_VALUE
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //将扩容后hashMap的临界值设置为newThr
    threshold = newThr;
    //创建新的table，初始化容量为newCap
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //修改hashMap的table为新建的newTab
    table = newTab;
    //如果旧table不为空，将旧table中的元素复制到新的table中
    if (oldTab != null) {
        //遍历旧哈希表的每个桶，将旧哈希表中的桶复制到新的哈希表中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //如果旧桶不为null，使用e记录旧桶
            if ((e = oldTab[j]) != null) {
                //将旧桶置为null
                oldTab[j] = null;
                //如果旧桶中只有一个node
                if (e.next == null)
                    //将e也就是oldTab[j]放入newTab中e.hash & (newCap - 1)的位置
                    newTab[e.hash & (newCap - 1)] = e;
                //如果旧桶中的结构为红黑树
                else if (e instanceof TreeNode)
                    //将树中的node分离
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { //如果旧桶中的结构为链表。这段没有仔细研究
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //遍历整个链表中的节点
                    do {
                        next = e.next;
                        //
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

从代码中可以看到，扩容很耗性能。所以在使用HashMap的时候，先估算map的大小，初始化的时候给一个大致的数值，避免map进行频繁的扩容。

看完代码后，可以将resize的步骤总结为

1. 计算扩容后的容量，临界值。
2. 将hashMap的临界值修改为扩容后的临界值
3. 根据扩容后的容量新建数组，然后将hashMap的table的引用指向新数组。
4. 将旧数组的元素复制到table中。

### 3.3 get方法

#### get( Object key)

```java
/**
 * 返回指定的key映射的value，如果value为null，则返回null。
 *
 * @see #put(Object, Object)
 */
public V get(Object key) {
    Node<K,V> e;
    //如果通过key获取到的node为null，则返回null，否则返回node的value。getNode方法的实现就在下面。
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

从源码中可以看到，get(E e)可以分为三个步骤：

1. 通过hash(Object key)方法计算key的哈希值hash。
2. 通过getNode( int hash, Object key)方法获取node。
3. 如果node为null，返回null，否则返回node.value。

#### getNode( int hash, Object key)

```java
/**
 * 根据key的哈希值和key获取对应的节点
 * 
 * @param hash 指定参数key的哈希值
 * @param key 指定参数key
 * @return 返回node，如果没有则返回null
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //如果哈希表不为空，而且key对应的桶上不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //如果桶中的第一个节点就和指定参数hash和key匹配上了
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            //返回桶中的第一个节点
            return first;
        //如果桶中的第一个节点没有匹配上，而且有后续节点
        if ((e = first.next) != null) {
            //如果当前的桶采用红黑树，则调用红黑树的get方法去获取节点
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //如果当前的桶不采用红黑树，即桶中节点结构为链式结构
            do {
                //遍历链表，直到key匹配
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    //如果哈希表为空，或者没有找到节点，返回null
    return null;
}
```

getNode方法又可分为以下几个步骤：

1. 如果哈希表为空，或key对应的桶为空，返回null
2. 如果桶中的第一个节点就和指定参数hash和key匹配上了，返回这个节点。
3. 如果桶中的第一个节点没有匹配上，而且有后续节点 
   1. 如果当前的桶采用红黑树，则调用红黑树的get方法去获取节点
   2. 如果当前的桶不采用红黑树，即桶中节点结构为链式结构，遍历链表，直到key匹配
4. 找到节点返回null，否则返回null。

### 3.4 remove方法

#### remove( Object key)

```java
/**
 * 删除hashMap中key映射的node
 *
 * @param  key 参数key
 * @return 如果没有映射到node，返回null，否则返回对应的value。
 */
public V remove(Object key) {
    Node<K,V> e;
    //根据key来删除node。removeNode方法的具体实现在下面
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
```

从源码中可以看到，remove方法的实现可以分为三个步骤：

1. 通过hash(Object key)方法计算key的哈希值。
2. 通过removeNode方法实现功能。
3. 返回被删除的node的value。

#### removeNode( int hash, Object key, Object value,boolean matchValue, boolean movable)

```java
/**
 * Map.remove和相关方法的实现需要的方法
 * 删除node
 * 
 * @param hash key的哈希值
 * @param key 参数key
 * @param value 如果matchValue为true，则value也作为确定被删除的node的条件之一，否则忽略
 * @param matchValue 如果为true，则value也作为确定被删除的node的条件之一
 * @param movable 如果为false，删除node时不会删除其他node
 * @return 返回被删除的node，如果没有node被删除，则返回null（针对红黑树的删除方法）
 */
final Node<K,V> removeNode(int hash, Object key, Object value,boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //如果数组table不为空且key映射到的桶不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        //
        Node<K,V> node = null, e; K k; V v;
        //如果桶上第一个node的就是要删除的node
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //记录桶上第一个node
            node = p;
        else if ((e = p.next) != null) {//如果桶内不止一个node
            if (p instanceof TreeNode)//如果桶内的结构为红黑树
                //记录key映射到的node
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {//如果桶内的结构为链表
                do {//遍历链表，找到key映射到的node
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        //记录key映射到的node
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //如果得到的node不为null且(matchValue为false||node.value和参数value匹配)
        if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
            //如果桶内的结构为红黑树
            if (node instanceof TreeNode)
                //使用红黑树的删除方法删除node
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)//如果桶的第一个node的就是要删除的node
                //删除node
                tab[index] = node.next;
            else//如果桶内的结构为链表，使用链表删除元素的方式删除node
                p.next = node.next;
            //结构性修改次数+1
            ++modCount;
            //哈希表大小-1
            --size;
            afterNodeRemoval(node);
            //返回被删除的node
            return node;
        }
    }
    //如果数组table为空或key映射到的桶为空，返回null。
    return null;
}
```

看完代码后，可以将removeNode方法的步骤总结为

1. 如果数组table为空或key映射到的桶为空，返回null。
2. 如果key映射到的桶上第一个node的就是要删除的node，记录下来。
3. 如果桶内不止一个node，且桶内的结构为红黑树，记录key映射到的node。
4. 桶内的结构不为红黑树，那么桶内的结构就肯定为链表，遍历链表，找到key映射到的node，记录下来。
5. 如果被记录下来的node不为null，删除node，size-1被删除。
6. 返回被删除的node。

## 四、HashMap与Hashtable对比

> 从存储结构和实现来讲基本上都是相同的。它和HashMap的最大的不同是它是线程安全的，另外它不允许key和value为null。Hashtable是个过时的集合类，不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/hashmap%E5%92%8Chashtable%E7%9A%84%E5%8C%BA%E5%88%AB.jpg)

## 五、总结

在JDK8中HashMap的底层是：**数组+链表(散列表)+红黑树**

在散列表中有装载因子这么一个属性，当装载因子*初始容量小于散列表元素时，该散列表会再散列，扩容2倍！

装载因子的**默认值是0.75**，无论是初始大了还是初始小了对我们HashMap的性能都不好

- 装载因子初始值大了，可以减少散列表再散列(扩容的次数)，但同时会导致散列冲突的可能性变大(**散列冲突也是耗性能的一个操作，要得操作链表(红黑树)**！
- 装载因子初始值小了，可以减小散列冲突的可能性，但同时扩容的次数可能就会变多！

初始容量的**默认值是16**，它也一样，无论初始大了还是小了，对我们的HashMap都是有影响的：

- 初始容量过大，那么遍历时我们的速度就会受影响~
- 初始容量过小，散列表再散列(扩容的次数)可能就变得多，扩容也是一件非常耗费性能的一件事~

从源码上我们可以发现：HashMap并不是直接拿key的哈希值来用的，它会将key的哈希值的高16位进行异或操作，使得我们将元素放入哈希表的时候**增加了一定的随机性**。

还要值得注意的是：**并不是桶子上有8位元素的时候它就能变成红黑树，它得同时满足我们的散列表容量大于64才行的**~

## 参考

- https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484139&idx=1&sn=bb73ac07081edabeaa199d973c3cc2b0&chksm=ebd743eadca0cafc532f298b6ab98b08205e87e37af6a6a2d33f5f2acaae245057fa01bd93f4&scene=21###wechat_redirect
- https://blog.csdn.net/panweiwei1994/article/details/77244920







