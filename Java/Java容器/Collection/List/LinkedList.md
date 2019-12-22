# LinkedList
## 一、概述
- LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
- LinkedList 实现 List 接口，能对它进行队列操作。
- LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
- LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
- LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
- LinkedList 是非同步的。


![](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/LinkedList%E7%BB%A7%E6%89%BF%E7%BB%93%E6%9E%84%E5%9B%BE)


## 二、源码分析
### 2.1 定义

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/LinkedList%E5%AE%9A%E4%B9%89.PNG)

==AbstractSequentialList== 实现了get(int index)、set(int index, E element)、add(int index, E element) 和 remove(int index)这些函数。**这些接口都是随机访问List的**，LinkedList是双向链表；既然它继承于AbstractSequentialList，就相当于已经实现了“get(int index)这些接口”。

此外，我们若需要通过AbstractSequentialList自己实现一个列表，只需要扩展此类，并提供 listIterator() 和 size() 方法的实现即可。若要实现不可修改的列表，则需要实现列表迭代器的 hasNext、next、hasPrevious、previous 和 index 方法即可。

### 2.2 属性

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/LinkedList%E5%B1%9E%E6%80%A7%E6%BA%90%E7%A0%81%E6%88%AA%E5%9B%BE.PNG)

size是链表的长度，first和last指的是头尾指针。

### 2.3 内部类（Node）


```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```


### 2.4 构造函数

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/LinkedList%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95%E6%BA%90%E7%A0%81.PNG)

1. 构造一个空链表。
2. 根据指定集合C构造LinkedList。先构造一个空的LinkedList，在把集合C的所有元素添加。

### 2.5 操作链表的底层方法
#### 2.5.1 linkFirst(E e)
方法在表头添加指定元素e

```java
/**
 * 在表头添加元素
 */
private void linkFirst(E e) {
    //使节点f指向原来的头结点
    final Node<E> f = first;
    //新建节点newNode，节点的前指针指向null，后指针原来的头节点
    final Node<E> newNode = new Node<>(null, e, f);
    //头指针指向新的头节点newNode 
    first = newNode;
    //如果原来的头结点为null，更新尾指针，否则使原来的头结点f的前置指针指向新的头结点newNode
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

#### 2.5.2 linkLast(E e)
方法在表尾插入指定元素e

```java
/**
 * 在表尾插入指定元素e
 */
void linkLast(E e) {
    //使节点l指向原来的尾结点
    final Node<E> l = last;
    //新建节点newNode，节点的前指针指向l，后指针为null
    final Node<E> newNode = new Node<>(l, e, null);
    //尾指针指向新的头节点newNode
    last = newNode;
    //如果原来的尾结点为null，更新头指针，否则使原来的尾结点l的后置指针指向新的头结点newNode
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

#### 2.5.3 linkBefore( E e, Node succ)
方法在指定节点succ之前插入指定元素e

```java
/**
 * 在指定节点succ之前插入指定元素e。指定节点succ不能为null。
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    //获得指定节点的前驱
    final Node<E> pred = succ.prev;
    //新建节点newNode，前置指针指向pred，后置指针指向succ
    final Node<E> newNode = new Node<>(pred, e, succ);
    //succ的前置指针指向newTouch
    succ.prev = newNode;
    //如果指定节点的前驱为null，将newTouch设为头节点。否则更新pred的后置节点
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

#### 2.5.4 unlinkFirst( Node f)
方法删除并返回头结点f。

```java
/**
 * 删除头结点f，并返回头结点的值
 */
private E unlinkFirst( Node<E> f) {
    // assert f == first && f != null;
    // 保存头结点的值
    final E element = f.item;
    // 保存头结点指向的下个节点
    final Node<E> next = f.next;
    //头结点的值置为null
    f.item = null;
    //头结点的后置指针指向null
    f.next = null; // help GC
    //将头结点置为next
    first = next;
    //如果next为null，将尾节点置为null，否则将next的后置指针指向null
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    //返回被删除的头结点的值
    return element;
}
```

#### 2.5.5 unlinkLast( Node l)
方法删除并返回尾结点f。

#### 2.5.6 unlink( Node x)
方法删除指定节点，返回指定元素的值x。


### 2.5 add方法
#### 2.6.1 add(E e) 方法
将元素添加到链表尾部

```java
public boolean add(E e) {
        linkLast(e);//这里就只调用了这一个方法
        return true;
    }
/**
 * 链接使e作为最后一个元素。
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;//新建节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;//指向后继元素也就是指向下一个元素
    size++;
    modCount++;
}
```
add方法的本质还是使用了linkLast函数实现的添加操作。

#### 2.6.2 add(int index,E e)
在指定位置添加元素

```java
public void add(int index, E element) {
    checkPositionIndex(index); //检查索引是否处于[0-size]之间

    if (index == size)//添加在链表尾部
        linkLast(element);
    else//添加在链表中间
        linkBefore(element, node(index));
}
```
linkBefore方法需要给定两个参数，一个插入节点的值，一个指定的node，所以我们又调用了Node(index)去找到index对应的node


#### 2.6.3 addAll(Collection c)
将集合插入到链表尾部

```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
```

#### 2.6.4 addAll(int index, Collection c)
将集合从指定位置开始插入

```java
public boolean addAll(int index, Collection<? extends E> c) {
    //1:检查index范围是否在size之内
    checkPositionIndex(index);

    //2:toArray()方法把集合的数据存到对象数组中
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    //3：得到插入位置的前驱节点和后继节点
    Node<E> pred, succ;
    //如果插入位置为尾部，前驱节点为last，后继节点为null
    if (index == size) {
        succ = null;
        pred = last;
    }
    //否则，调用node()方法得到后继节点，再得到前驱节点
    else {
        succ = node(index);
        pred = succ.prev;
    }

    // 4：遍历数据将数据插入
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        //创建新节点
        Node<E> newNode = new Node<>(pred, e, null);
        //如果插入位置在链表头部
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    //如果插入位置在尾部，重置last节点
    if (succ == null) {
        last = pred;
    }
    //否则，将插入的链表与先前链表连接起来
    else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}    
```

## 2.6 根据位置取数据的方法
### 2.6.1 get(int index)
根据指定索引返回数据


```java
public E get(int index) {
    //检查index范围是否在size之内
    checkElementIndex(index);
    //调用Node(index)去找到index对应的node然后返回它的值
    return node(index).item;
}
```

### 2.6.2 获取头节点（index=0）数据方法

```java
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
public E element() {
    return getFirst();
}
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```
**区别**： getFirst(),element(),peek(),peekFirst() 这四个获取头结点方法的区别在于对链表为空时的处理，是抛出异常还是返回null，其中getFirst() 和element() 方法将会在链表为空时，抛出异常

element()方法的内部就是使用getFirst()实现的。它们会在链表为空时，抛出NoSuchElementException

### 2.6.3  获取尾节点（index=-1）数据方法

```java
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
 public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```
**两者区别**： getLast() 方法在链表为空时，会抛出**NoSuchElementException**，而peekLast() 则不会，只是会返回 null。

## 2.7 根据对象得到索引的方法
### 2.7.1 int indexOf(Object o)
从头遍历找

```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        //从头遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        //从头遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

### 2.7.2 int lastIndexOf(Object o)
从尾遍历找

```java
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        //从尾遍历
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        //从尾遍历
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

## 2.8 检查链表是否包含某对象的方法
### 2.8.1 contains(Object o)
检查对象o是否存在于链表中

```java
public boolean contains(Object o) {
    return indexOf(o) != -1;
}
```

## 2.9 删除方法
### 2.9.1 remove() ,removeFirst(),pop()
删除头节点

```java
public E pop() {
    return removeFirst();
}
public E remove() {
    return removeFirst();
}
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```

### 2.9.2 removeLast(),pollLast()
删除尾节点

```java
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
```
**区别**： removeLast()在链表为空时将抛出NoSuchElementException，而pollLast()方法返回null。