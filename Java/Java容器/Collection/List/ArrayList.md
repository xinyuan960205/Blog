# ArrayList
## 一、概述

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ArrayList%E7%BB%A7%E6%89%BF%E5%9B%BE.webp)

- ArrayList 的底层是数组队列，相当于动态数组。与 Java 中的数组相比，它的容量能动态增长。在添加大量元素前，应用程序可以使用`ensureCapacity`操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量。

- 它继承于**AbstractList**，实现了 **List, RandomAccess, Cloneable, java.io.Serializable**这些接口。

- 在我们学数据结构的时候就知道了线性表的顺序存储，插入删除元素的时间复杂度为**O（n）**,求表长以及增加元素，取第 i 元素的时间复杂度为**O（1）**

- ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。

- ArrayList 实现了RandomAccess 接口， RandomAccess 是一个标志接口，表明实现这个接口的 List 集合是支持快速随机访问的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。

- ArrayList 实现了Cloneable 接口，即覆盖了函数 clone()，能被克隆。

- ArrayList 实现java.io.Serializable 接口，这意味着ArrayList支持序列化，能通过序列化去传输。

- 和 Vector 不同，**ArrayList中的操作不是线程安全的**！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

## 二、源码分析
### 2.1 属性

- 截图分析
  ![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ArrayList%E5%B1%9E%E6%80%A7.webp)


- 底层是数组

根据上面我们可以清晰的发现：**==ArrayList底层其实就是一个数组==**，ArrayList中有扩容这么一个概念，正因为它扩容，所以它能够 **==实现“动态”增长==**.

- 源码：
```java
private static final long serialVersionUID = 8683452581122892189L;

/**
 * 默认初始容量大小
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * 空数组（用于空实例）。
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

 //用于默认大小空实例的共享空数组实例。
  //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 保存ArrayList数据的数组
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * ArrayList 所包含的元素个数
 */
private int size;
```

### 2.2 构造函数
- 截图分析

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ArrayList%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.webp)

- 三个构造函数

1. ArrayList()：默认构造函数，提供初始容量为 10 的空列表。
2. ArrayList(int initialCapacity)：构造一个具有指定初始容量的空列表。
3. ArrayList(Collection<? extends E> c)：构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。

- 源码：

```java
/**
 * 带初始容量参数的构造函数。（用户自己指定容量）
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        //创建initialCapacity大小的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        //创建空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

/**
 *默认构造函数，DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
 */
public ArrayList(Collection<? extends E> c) {
    //
    elementData = c.toArray();
    //如果指定集合元素个数不为0
    if ((size = elementData.length) != 0) {
        // c.toArray 可能返回的不是Object类型的数组所以加上下面的语句用于判断，
        //这里用到了反射里面的getClass()方法
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 用空数组代替
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

### 2.3 新增（add）
add方法：

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ArrayList%20Add%E6%96%B9%E6%B3%95.webp)

#### 2.3.1 add(E e)
- 源码分析：

```java
    /**
     * 将指定的元素追加到此列表的末尾。 
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }

```
这里 ensureCapacityInternal() 方法是对 ArrayList 集合进行扩容操作。
- 步骤：
1. 检查是否需要扩容
2. 插入元素

#### 2.3.2 add(int index, E element)
- 源码分析：

```java
/**
 * 在此列表中的指定位置插入指定的元素。 
 *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
 *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
 */
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //arraycopy()这个实现数组之间复制的方法一定要看一下，下面就用到了arraycopy()方法实现数组自己复制自己
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```
- 步骤：
1. 检查角标
2. 空间检查，如果有需要进行扩容
3. 插入元素

### 2.4 扩容
#### 2.4.1 扩容步骤：
1. add方法先调用`ensureCapacityInternal`
- 确认list容量，尝试容量加1，看看有无必要
- 添加元素

接下来我们来看看这个小容量(+1)是否满足我们的需求：

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ensureCapacityInternal%E5%87%BD%E6%95%B0%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.webp)

2. 随后调用ensureExplicitCapacity()来确定明确的容量

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ensureExplicitCapacity%E5%87%BD%E6%95%B0%E6%BA%90%E7%A0%81.webp)

3. 接下来看看grow()

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/grow()%E5%87%BD%E6%95%B0%E6%BA%90%E7%A0%81.webp)

4. 进去看copyOf()方法

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/copyOf()%E5%87%BD%E6%95%B0%E6%BA%90%E7%A0%81.webp)

#### 2.4.2 源码分析：
```java
//下面是ArrayList的扩容机制
//ArrayList的扩容机制提高了性能，如果每次只扩充一个，
//那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
/**
 * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
 * @param   minCapacity   所需的最小容量
 */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
//得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
          // 获取默认的容量和传入参数的较大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
//判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        //调用grow方法进行扩容，调用此方法代表已经开始扩容了
        grow(minCapacity);
}

/**
 * ArrayList扩容的核心方法。
 */
private void grow(int minCapacity) {
    // oldCapacity为旧容量，newCapacity为新容量
    int oldCapacity = elementData.length;
    //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
    //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //再检查新容量是否超出了ArrayList所定义的最大容量，
    //若超出了，则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE，
    //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

#### 2.4.3 扩容总结
- 首先去检查一下数组的容量是否足够
- 扩容到原来的1.5倍
- 第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。
- 足够：直接添加；不足够：扩容

- 另外需要注意的是：
1. java 中的length 属性是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
2. java 中的length()方法是针对字符串String说的,如果想看这个字符串的长度则用到 length()这个方法.
3. java 中的size()方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!

- 移位运算符

    另外值得注意的是大家很容易忽略的一个运算符：移位运算符 　　简介：移位运算符就是在二进制的基础上对数字进行平移。按照平移的方向和填充数字的规则分为三种:<<(左移)、>>(带符号右移)和>>>(无符号右移)。 　　作用：对于大数据的2进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源 　　比如这里：int newCapacity = oldCapacity + (oldCapacity >> 1); 右移一位相当于除2，右移n位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了1位所以相当于oldCapacity /2。

### 2.5 删除（remove）
#### 2.5.1 步骤
1. 检查角标
2. 删除元素
3. 计算出需要移动的个数，并移动
4. 设置为null，让Gc回收

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/remove%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.webp)

#### 2.5.2 源码

```java
/**
 * 删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）。 
 */
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
  //从列表中删除的元素 
    return oldValue;
}

/**
 * 从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。
 *返回true，如果此列表包含指定的元素
 */
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

/*
 * Private remove method that skips bounds checking and does not
 * return the value removed.
 */
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

/**
 * 从列表中删除所有元素。 
 */
public void clear() {
    modCount++;

    // 把数组中所有的元素的值设为null
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```


### 2.6 查找（get）
#### 2.6.1 步骤
1. 检查角标
2. 返回元素

#### 2.6.2 源码分析

```java
/**
 * 返回此列表中指定位置的元素。
 */
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}

// 检查角标
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

// 返回元素
E elementData(int index) {
    return (E) elementData[index];
}
```
## 三、细节再说明(trimToSize)
- ArrayList是**基于动态数组实现的**，在增删时候，需要数组的拷贝复制。
- **ArrayList的默认初始化容量是10，每次扩容时候增加原先容量的一半，也就是变为原来的1.5倍。**
- 删除元素时不会减少容量，**若希望减少容量则调用trimToSize()**。
- 它不是线程安全的。它能存放null值。

## 四、Fail-Fast
modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

### 安全失败Fail-safe

 采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

   原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

   缺点：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。

​     场景：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。

## 五、序列化

ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。

```java
transient Object[] elementData; // non-private to simplify nested class access
```

ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去反射调用该对象的 writeObject() 来实现序列化。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
```

## 六、ArrayList使用
### 6.1 toArray()的使用
有时候，当我们调用ArrayList中的 toArray()，可能遇到过抛出java.lang.ClassCastException异常的情况，这是由于toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(如，将Object[]转换为的Integer[])则会抛出java.lang.ClassCastException异常，因为Java不支持向下转型。
所以一般更常用的是使用另外一种方法进行使用：
```java
<T> T[] toArray(T[] a)
```

调用toArray(T[] a)返回T[]可通以下方式进行实现：

```java
 // toArray用法
 // 第一种方式(最常用)
 Integer[] integer = arrayList.toArray(new Integer[0]);

 // 第二种方式(容易理解)
 Integer[] integer1 = new Integer[arrayList.size()];
 arrayList.toArray(integer1);

 // 抛出异常，java不支持向下转型
 //Integer[] integer2 = new Integer[arrayList.size()];
 //integer2 = arrayList.toArray();
```