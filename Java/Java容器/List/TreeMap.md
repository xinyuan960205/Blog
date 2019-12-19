# TreeMap

## 一、概述

TreeMap是Map接口基于红黑树的实现，键值对是有序的。本文主要讲解关于红黑树实现的部分。

## 二、源码解析

### 2.1 顶部注释

> A Red-Black tree based NavigableMap implementation. The map is sorted according to the Comparable natural ordering of its keys, or by a Comparator provided at map creation time, depending on which constructor is used.
>

TreeMap是基于NavigableMap的红黑树实现。TreeMap根据键的自然顺序进行排序，或者根据创建map时提供的Comparator进行排序，使用哪种取决于使用的哪个构造方法。

> This implementation provides guaranteed log(n) time cost for the containsKey,get, put and remove operations. Algorithms are adaptations of those in Cormen, Leiserson, and Rivest’s Introduction to Algorithms.
>

TreeMap提供时间复杂度为log(n)的containsKey，get，put ，remove操作。

下面的注释主要讲TreeMap是非同步的，它的迭代器是fail-fastl的。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TreeMap%E9%A1%B6%E9%83%A8%E8%A7%A3%E6%9E%90.webp)

顶部注释总结：

- TreeMap实现了NavigableMap接口，而NavigableMap接口继承着SortedMap接口，致使我们的**TreeMap是有序的**！
- TreeMap底层是红黑树，它方法的时间复杂度都不会太高:log(n)~
- 非同步
- 使用Comparator或者Comparable来比较key是否相等与排序的问题~

### 2.2 类结构图

```java
public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

从中我们可以了解到：

- TreeMap<K,V>：TreeMap是以key-value形式存储数据的。
- extends AbstractMap<K,V>：继承了AbstractMap，实现Map接口时需要实现的工作量大大减少了。
- implements NavigableMap：实现了NavigableMap，可以返回特定条件最近匹配的导航方法。
- implements Cloneable：表明其可以调用clone()方法来返回实例的field-for-field拷贝。
- implements Serializable：表明该类是可以序列化的。

结构图：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TreeMap%E7%B1%BB%E7%BB%A7%E6%89%BF%E5%9B%BE.webp)

### 2.3 TreeMap的域

```java
/**
 * treeMap的排序规则，如果为null，则根据键的自然顺序进行排序
 *
 * @serial
 */
private final Comparator<? super K> comparator;

/**
 * 红黑数的根节点
 */
private transient Entry<K,V> root;

/**
 * 红黑树节点的个数
 */
private transient int size = 0;

/**
 * treeMap的结构性修改次数。实现fast-fail机制的关键。
 */
private transient int modCount = 0;
```

### 2.4 构造函数

可以发现，TreeMap的构造方法大多数与comparator有关。

也就是顶部注释说的：TreeMap有序是通过Comparator来进行比较的，**如果comparator为null，那么就使用自然顺序**~

#### 2.4.1 TreeMap()

```java
/**
 * 
 * 使用key的自然排序来构造一个空的treeMap
 */
public TreeMap() {
    comparator = null;
}
```

#### 2.4.2 TreeMap(Comparator<? super K> comparator)

```java
/**
 * 使用给定的比较器来构造一个空的treeMap
 *
 * @param 给定的比较器
 */
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
```

#### 2.4.3 TreeMap(Map<? extends K, ? extends V> m)

```java
/**
 * 使用key的自然排序来构造一个treeMap，treeMap包含给定map中所有的键值对
 *
 * @param  m 给定map
 * @throws ClassCastException 如果map中的key不是Comparable或者不是相互可比较的
 * @throws NullPointerException 如果参数map为null
 */
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}
```

关于putAll()方法的讲解请往下看。

putAll( Map<? extends K, ? extends V> map)

```java
/**
 * 将参数map中的所有键值对映射插入到hashMap中，如果有碰撞，则覆盖value。
 *
 * @param  map 参数map
 * @throws ClassCastException
 * @throws NullPointerException 参数map为null或者参数map中含有值为null的key且treeMap不允许有值为null的key
 */
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();
    //如果treeMap大小为0且参数map大小不为0且参数map是有序的
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        //取出参数map的比较器
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        //如果参数map的比较器等于treeMap的比较器
        if (c == comparator || (c != null && c.equals(comparator))) {
            //结构性修改次数+1
            ++modCount;
            try {
                //根据已经一个排好序的map创建一个TreeMap。该方法将map中的元素逐个添加到TreeMap中，并返回map的中间元素作为根节点。
                buildFromSorted(mapSize, map.entrySet().iterator(),null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }
    //如果执行到了这一步，其实是按照map的entrySet的迭代器的顺序将参数map中所有键值对复制到treeMap中的
    super.putAll(map);
}
```

#### TreeMap(SortedMap<K, ? extends V> m)

```java
/**
 * 使用指定的sortedMap来构造treeMap。treeMap中含有sortedMap中所有的键值对，键值对顺序和sortedMap中相同。该方法时间复杂度是线性的。
 *
 * @param  m 指定的sortedMap
 * @throws NullPointerException 如果指定的sortedMap为null
 */
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

该构造函数不同于上一个构造函数TreeMap(Map<? extends K, ? extends V> m)，在上一个构造函数中传入的参数是Map，Map不一定有序的。而本构造函数的参数是SortedMap类型，是有序的。

关于buildFromSorted方法的讲解请往下看。

### 2.5 put方法

#### 2.5.1 put( K key, V value)

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TreeMap%E7%9A%84put%E6%96%B9%E6%B3%95.webp)

```java
/**
 * 将指定参数key和指定参数value插入map中，如果key已经存在，那就替换key对应的value
 *
 * @param key 参数key
 * @param value 参数value
 *
 * @return 如果value被替换，则返回旧的value，否则返回null。当然，可能key对应的value就是null。
 * @throws ClassCastException 如果指定参数key不能和map中的key比较
 * @throws NullPointerException 如果指定参数key为null而且map使用自然排序，或者比较器不允许key为null
 */
public V put(K key, V value) {
    Entry<K,V> t = root;
    //如果根节点为空    
    if (t == null) {
        compare(key, key); // 对key是否为null进行检查type

        //创建一个根节点，返回null
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    //记录比较结果
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    //如果comparator不为null
    if (cpr != null) {
        //循环查找key要插入的位置
        do {
            //记录上次循环的节点t
            parent = t;
            //比较节点t的key和参数key的大小
            cmp = cpr.compare(key, t.key);
            //如果节点t的key > 参数key
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)//如果节点t的key < 参数key
                t = t.right;
            else//如果节点t的key = 参数key，替换value，返回旧value
                return t.setValue(value);
        } while (t != null);//t为null，没有要比较的节点，代表已经找到新节点要插入的位置
    }
    else {如果comparator为null，，则使用key作为比较器进行比较，并且key必须实现Comparable接口
        //如果key为null，抛出NullPointerException
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {//循环查找key要插入的位置
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 找到新节点的父节点后，创建节点对象
    Entry<K,V> e = new Entry<>(key, value, parent);
    //如果新节点key的值小于父节点key的值，则插在父节点的左侧
    if (cmp < 0)
        parent.left = e;
    else//否则插在父节点的左侧
        parent.right = e;
    //插入新的节点后，为了保持红黑树平衡，对红黑树进行调整
    fixAfterInsertion(e);
    size++;
    modCount++;
    //这种情况下没有替换旧value，返回努力了
    return null;
}
```

下面是`compare(Object k1, Object k2)`方法

```java
    /**
     * Compares two keys using the correct comparison method for this TreeMap.
     */
    @SuppressWarnings("unchecked")
    final int compare(Object k1, Object k2) {
        return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
            : comparator.compare((K)k1, (K)k2);
    }
```

如果我们设置key为null，会抛出异常的，就不执行下面的代码了。

### 2.6 get方法

实际上就是如何自己实现了comparator，就用自己实现的，不然就用默认的。

#### get(Object key)

```java
/**
 * 返回指定的key对应的value，如果value为null，则返回null
 *
 * @throws ClassCastException 如果参数key和treeMap中的key不可比较
 * @throws NullPointerException 如果参数key为null而且treeMap使用自然排序或者比较器不允许key为null
 */
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}
```

#### getEntry( Object key)

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/getEntry.webp)

```java
/**
 * 返回参数key对应的节点
 *
 * @return 返回节点，如果没有则返回null
 * @throws ClassCastException 如果参数key和treeMap中的key无法比较
 * @throws NullPointerException 如果参数key为null而且treeMap使用自然排序或者比较器不允许key为null
 */
final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    //如果比较器为不为null
    if (comparator != null)
        //通过比较器来获取结果。下个介绍的方法就是getEntryUsingComparator
        return getEntryUsingComparator(key);
    //如果key为null，抛出NullPointerException
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    //按照二叉树搜索的方式进行搜索，搜到返回
    while (p != null) {
        //比较节点的key和参数key
        int cmp = k.compareTo(p.key);
        //如果节点的key小于参数key
        if (cmp < 0)
            //向左遍历
            p = p.left;
        else if (cmp > 0)//如果节点的key大于参数key
            //向左遍历
            p = p.right;
        else//如果节点的key等于参数key
            return p;
    }
    //如果遍历完依然没有找到对应的节点，返回null
    return null;
}
```

#### getEntryUsingComparator( Object key)

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/getEntryUsingComparator.webp)

```java
/**
 * 使用comparator获取节点。
 */
final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
        K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        Entry<K,V> p = root;
        //按照二叉树搜索的方式进行搜索，搜到返回
        while (p != null) {
        //比较节点的key和参数key
        int cmp = k.compareTo(p.key);
        //如果节点的key小于参数key
        if (cmp < 0)
            //向左遍历
            p = p.left;
        else if (cmp > 0)//如果节点的key大于参数key
            //向左遍历
            p = p.right;
        else//如果节点的key等于参数key
            return p;
        }
    }
    return null;
}
```

### 2.7 remove方法

#### remove( Object key)

```java
/**
 * 如果key在treeMap中存在，删除对应的节点，返回旧value，否则返回null
 *
 * @param  key 参数key
 * @return 如果节点被删除，返回节点的value，否则返回null。当然，可能key对应的value就是null
 * @throws ClassCastException 如果指定参数key为null而且map使用自然排序，或者比较器不允许key为null
 */
public V remove(Object key) {
    //获取key对应的节点
    Entry<K,V> p = getEntry(key);
    //如果节点为null，返回null
    if (p == null)
        return null;
    //记录旧value
    V oldValue = p.value;
    //删除节点
    deleteEntry(p);
    //返回旧value
    return oldValue;
}
```

删除节点的时候调用的是`deleteEntry(Entry<K,V> p)`方法，这个方法主要是**删除节点并且平衡红黑树**

### 2.8 遍历方法

遍历的方法有非常多。

debug一下看看，于是乎，我们可以找到：**TreeMap遍历是使用EntryIterator这个内部类的**。

于是乎，我们可以找到：**TreeMap遍历是使用EntryIterator这个内部类的**

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/EntryIterator%E7%B1%BB%E7%BB%93%E6%9E%84%E5%9B%BE.webp)

那接下来我们去看看PrivateEntryIterator比较重要的方法：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/EntryIterator%E6%BA%90%E7%A0%81.PNG)

我们进去`successor(e)`方法看看实现：

> successor 其实就是一个结点的 下一个结点，所谓 下一个，是按次序排序后的下一个结点。从代码中可以看出，如果右子树不为空，就返回右子树中最小结点。如果右子树为空，就要向上回溯了。在这种情况下，t 是以其为根的树的最后一个结点。如果它是其父结点的左孩子，那么父结点就是它的下一个结点，否则，t 就是以其父结点为根的树的最后一个结点，需要再次向上回溯。一直到 ch 是 p 的右孩子为止。

```java
    /**
     * Returns the successor of the specified Entry, or null if no such.
     */
    static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
        if (t == null)
            return null;
        else if (t.right != null) {
            Entry<K,V> p = t.right;
            while (p.left != null)
                p = p.left;
            return p;
        } else {
            Entry<K,V> p = t.parent;
            Entry<K,V> ch = t;
            while (p != null && ch == p.right) {
                ch = p;
                p = p.parent;
            }
            return p;
        }
    }
```

## 三、总结

TreeMap底层是红黑树，能够实现该Map集合有序~

如果在构造方法中传递了Comparator对象，那么就会以Comparator对象的方法进行比较。否则，则使用Comparable的`compareTo(T o)`方法来比较。

- 值得说明的是：如果使用的是`compareTo(T o)`方法来比较，**key一定是不能为null**，并且得实现了Comparable接口的。
- 即使是传入了Comparator对象，不用`compareTo(T o)`方法来比较，key**也是**不能为null的

```java
    public static void main(String[] args) {
        TreeMap<Student, String> map = new TreeMap<Student, String>((o1, o2) -> {
            //主要条件
            int num = o1.getAge() - o2.getAge();

            //次要条件
            int num2 = num == 0 ? o1.getName().compareTo(o2.getName()) : num;

            return num2;
        });

        //创建学生对象
        Student s1 = new Student("潘安", 30);
        Student s2 = new Student("柳下惠", 35);

        //添加元素进集合
        map.put(s1, "宋朝");
        map.put(s2, "元朝");
        map.put(null, "汉朝");

        //获取key集合
        Set<Student> set = map.keySet();

        //遍历key集合
        for (Student student : set) {
            String value = map.get(student);
            System.out.println(student + "---------" + value);
        }
    }
```

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib1licMOAH8TWNOKlvRRDyCiaFQgLptPmQltrmDiaPNe68kKSUkxT5ehrCplLIODlDeB7uxCmIeU2FnkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们从源码中的很多地方中发现：Comparator和Comparable出现的频率是很高的，因为TreeMap实现有序要么就是外界传递进来Comparator对象，要么就使用默认key的Comparable接口(实现自然排序)

最后我就来总结一下TreeMap要点吧：

1. 由于底层是红黑树，那么时间复杂度可以保证为log(n)
2. key不能为null，为null为抛出NullPointException的
3. 想要自定义比较，在构造方法中传入Comparator对象，否则使用key的自然排序来进行比较
4. TreeMap非同步的，想要同步可以使用Collections来进行封装



## 参考

- <https://blog.csdn.net/panweiwei1994/article/details/77530819>
- <https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484147&idx=1&sn=1381f3cf767ab5184369e7729abd6139&chksm=ebd743f2dca0cae46c7fe8c704deff18b263245c5e4b4376c182cc8061f62ac61288b06d4226&scene=21###wechat_redirect>
- 

