# Set

## 一、HashSet

### 1.1 结构图

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/hashset%E7%BB%A7%E6%89%BF%E7%BB%93%E6%9E%84%E5%9B%BE.webp)

### 1.2 顶部注释

> 此类实现 Set 接口，由哈希表（实际上是一个 HashMap 实例）支持。它不保证set的迭代顺序；特别是它不保证该顺序恒久不变。此类允许使用 null 元素。
>
> 此类为基本操作提供了稳定性能，这些基本操作包括 add、remove、contains 和 size，假定哈希函数将这些元素正确地分布在桶中。对此set进行迭代所需的时间与 HashSet 实例的大小（元素的数量）和底层 HashMap 实例（桶的数量）的“容量”的和成比例。因此，如果迭代性能很重要，则不要将初始容量设置得太高（或将加载因子设置得太低）。
>
> 注意，此实现不是同步的。如果多个线程同时访问一个哈希 set，而其中至少一个线程修改了该 set，那么它必须 保持外部同步。这通常是通过对自然封装该set的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSet 方法来“包装” set。最好在创建时完成这一操作，以防止对该set进行意外的不同步访问：
> Set s = Collections.synchronizedSet(new HashSet(…));
>
> 此类的 iterator 方法返回的迭代器是fail-fast的：在创建迭代器之后，如果对set进行修改，除非通过迭代器自身的 remove 方法，否则在任何时间以任何方式对其进行修改，Iterator 都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒将来在某个不确定时间发生任意不确定行为的风险。
>
> 注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器在尽最大努力抛出 ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误做法：迭代器的快速失败行为应该仅用于检测 bug。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/hashset%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.webp)

从顶部注释来看，我们就可以归纳HashSet的要点了：

- 实现Set接口
- 不保证迭代顺序
- 允许元素为null
- **底层实际上是一个HashMap实例**
- 非同步
- 初始容量非常影响迭代性能

### 1.3 域与定义

```java
/**
 * 说明HashSet是依赖于HashMap的，底层就是一个HashMap实例。
 */
private transient HashMap<E,Object> map;
/**
 * 前面讲了，HashSet是依赖于HashMap的，底层就是一个HashMap实例。HashMap是保存键值对的，但我们保存hashSet的时候肯定只是想保存key，那么调用hashMap(key,value)时value应该传什么值呢？PRESENT就是value。
 */
private static final Object PRESENT = new Object();

/*
 * 构造方法之一，其他几个构造方法也是类似的。
 */
public HashSet() {
    map = new HashMap<>();
}
```

HashSet实际上就是封装了HashMap，**操作HashSet元素实际上就是操作HashMap**。这也是面向对象的一种体现，**重用性贼高**！

HashMap的key就是HashSet的值，而HashMap的value就是`PRESENT`

## 二、TreeSet

### 2.1 结构图

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TreeSet%E7%BB%93%E6%9E%84%E5%9B%BE.webp)

### 2.2 顶部注释

> 基于TreeMap的NavigableSet实现。使用元素的自然顺序对元素进行排序，或者根据创建set时提供的Comparator进行排序，具体取决于使用的构造方法。
>
> 此实现为基本操作（add、remove和contains）提供受保证的log(n)时间开销。
>
> 注意，如果要正确实现Set接口，则set维护的顺序（无论是否提供了显式比较器）必须与equals一致。（关于与equals一致的精确定义，请参阅Comparable或Comparator。）这是因为Set接口是按照equals操作定义的，但TreeSet实例使用它的compareTo（或compare）方法对所有元素进行比较，因此从set的观点来看，此方法认为相等的两个元素就是相等的。即使set的顺序与equals不一致，其行为也是定义良好的；它只是违背了Set接口的常规协定。
>
> 注意，此实现不是同步的。如果多个线程同时访问一个TreeSet，而其中至少一个线程修改了该set，那么它必须外部同步。这一般是通过对自然封装该set的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用Collections.synchronizedSortedSet方法来“包装”该set。此操作最好在创建时进行，以防止对set的意外非同步访问：
> SortedSet s= Collections.synchronizedSortedSet(new TreeSet(…));
>
> 此类的iterator方法返回的迭代器是fail-fast的：在创建迭代器之后，如果从结构上对set进行修改（除非通过迭代器自身的remove方法），否则在其他任何时间以任何方式进行修改都将导致迭代器抛出ConcurrentModificationException。因此，对于并发的修改，迭代器很快就完全失败，而不会冒着在将来不确定的时间发生不确定行为的风险。
>
> 注意，迭代器的fail-fast行为无法得到保证，一般来说，存在不同步的并发修改时，不可能作出任何肯定的保证。fail-fast迭代器尽最大努力抛出ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测bug。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TreeSet%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.webp)

从顶部注释来看，我们就可以归纳TreeSet的要点了：

- 实现NavigableSet接口
- 可以实现排序功能
- **底层实际上是一个TreeMap实例**
- 非同步

### 2.3 域与定义

```java
/**
 * The backing map.
 */
private transient NavigableMap<E,Object> m;

/**
 *  前面讲了，TreeSet是依赖于TreeMap的，底层就是一个TreeMap实例。TreeMap是保存键值对的，但我们保存TreeSet的时候肯定只是想保存key，那么调用hashMap(key,value)时value应该传什么值呢？PRESENT就是要传value。
 */
private static final Object PRESENT = new Object();

private transient NavigableMap<E,Object> m;

TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}

public TreeSet() {
    this(new TreeMap<E,Object>());
}
```

## 三、LinkedHashSet

### 3.1 结构图

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/LinkedHashSet%E7%BB%93%E6%9E%84%E5%9B%BE111.webp)

### 3.2 顶部注释

从顶部注释来看，我们就可以归纳LinkedHashSet的要点了：

- 迭代是有序的
- 允许为null
- **底层实际上是一个HashMap+双向链表实例(其实就是LinkedHashMap)…**
- 非同步
- 性能比HashSet差一丢丢，因为要维护一个双向链表
- 初始容量与迭代无关，LinkedHashSet迭代的是双向链表

## 四、总结

可以很明显地看到，**Set集合的底层就是Map**，所以我都没有做太多的分析在上面，也没什么好分析的了。

下面总结一下Set集合常用的三个子类吧：

**HashSet：**

- 无序，允许为null，底层是HashMap(散列表+红黑树)，非线程同步

**TreeSet：**

- 有序，不允许为null，底层是TreeMap(红黑树),非线程同步

**LinkedHashSet：**

- 迭代有序，允许为null，底层是HashMap+双向链表，非线程同步

## 参考

- https://zhuanlan.zhihu.com/p/29021276
- https://blog.csdn.net/panweiwei1994/article/details/76555359