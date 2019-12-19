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





