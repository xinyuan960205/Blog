# Collection
## 一、集合（Collection）
### 1.1 为什么需要集合
1. Java是一门面向对象的语言，就免不了处理对象
2. 为了方便操作多个对象，那么我们就得把这多个对象存储起来
3. 想要存储多个对象(变量),很容易就能想到一个容器
4. 常用的容器我们知道有-->StringBuffered,数组(虽然有对象数组，但是数组的长度是不可变的！)
5. 所以，Java就为我们提供了集合(Collection)～

### 1.2 数组和集合的区别
1. 长度的区别
- 数组的长度固定
- 集合的长度可变
2. 内容不同
- 数组存储的是同一种类型的元素
- 集合可以存储不同类型的元素(但是一般我们不这样干..)
3. 元素的数据类型
- 数组可以存储基本数据类型,也可以存储引用类型
- 集合只能存储引用类型(你存储的是简单的int，它会自动装箱成Integer)

### Collection体系结构
![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/collection%E7%BB%93%E6%9E%84%E5%9B%BE.webp)

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/collection%E6%96%B9%E6%B3%95%E7%BB%93%E6%9E%84.PNG)

## 二、Set
Set集合的特点是：元素不可重复

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/Set%E7%BB%93%E6%9E%84.PNG)

Set集合常用子类
- HashSet集合
    - A:底层数据结构是哈希表(是一个元素为链表的数组)
- TreeSet集合
    - A:底层数据结构是红黑树(是一个自平衡的二叉树)
    - B:保证元素的排序方式
- LinkedHashSet集合
    - A:：底层数据结构由哈希表和链表组成

## 三、List
List集合的特点就是：有序(存储顺序和取出顺序一致),可重复
![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/List%E7%BB%93%E6%9E%84.PNG)

List集合常用的子类有三个：
- ArrayList
    - 底层数据结构是数组。线程不安全
- LinkedList
    - 底层数据结构是链表。线程不安全
- Vector
    - 底层数据结构是数组。线程安全

## 四、Queue
Queue用户模拟队列这种数据结构，队列通常是指“先进先出”(FIFO，first-in-first-out)的容器。队列的头部是在队列中存放时间最长的元素，队列的尾部是保存在队列中存放时间最短的元素。新元素插入（offer）到队列的尾部，访问元素（poll）操作会返回队列头部的元素。通常，队列不允许随机访问队列中的元素。

## 五、迭代器（Iterator）
### 5.1 概要
为了方便的处理集合中的元素,Java中出现了一个对象,该对象提供了一些方法专门处理集合中的元素.例如删除和获取集合中的元素.该对象就叫做迭代器(Iterator).

对 Collection 进行迭代的类，称其为迭代器。还是面向对象的思想，专业对象做专业的事情，迭代器就是专门取出集合元素的对象。但是该对象比较特殊，不能直接创建对象（通过new），该对象是以内部类的形式存在于每个集合类的内部。

如何获取迭代器？Collection接口中定义了获取集合类迭代器的方法（iterator（）），所以所有的Collection体系集合都可以获取自身的迭代器。

### 5.2 Iterable
Collection的源码中继承了Iterable，有iterator()这个方法

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/Iterable%E7%BB%93%E6%9E%84.PNG)

正是由于每一个容器都有取出元素的功能。这些功能定义都一样，只不过实现的具体方式不同（因为每一个容器的数据结构不一样）所以对共性的取出功能进行了抽取，从而出现了Iterator接口。而每一个容器都在其内部对该接口进行了内部类的实现。也就是将取出方式的细节进行封装。


Jdk1.5之后添加的新接口, Collection的父接口. 实现了Iterable的类就是可迭代的.并且支持增强for循环。该接口只有一个方法即获取迭代器的方法iterator（）可以获取每个容器自身的迭代器Iterator。（Collection）集合容器都需要获取迭代器（Iterator）于是在5.0后又进行了抽取将获取容器迭代器的iterator（）方法放入到了Iterable接口中。Collection接口进程了Iterable，所以Collection体系都具备获取自身迭代器的方法，只不过每个子类集合都进行了重写（因为数据结构不同）

### 5.3 Iterator 

再来看一下，Iterator也是一个接口，它只有三个方法：
- hasNext()
- next()
- remove()

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/Iterator%E7%BB%93%E6%9E%84.PNG)

### 5.4 迭代器的遍历使用
1. while循环

```Java

public class Demo1 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
		Iterator it = list.iterator();
		while (it.hasNext()) {
			String next = (String) it.next();
			System.out.println(next);
		}
	}
```

2. for循环

```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		for (Iterator it = list.iterator(); it.hasNext();) {
             //迭代器的next方法返回值类型是Object，所以要记得类型转换。
			String next = (String) it.next();
			System.out.println(next);
		}
	}
}
```
需要取出所有元素时，可以通过循环，java 建议使用for 循环。因为可以对内存进行一下优化。

3. 使用迭代器清空集合

```java
public class Demo1 {
	public static void main(String[] args) {
		Collection coll = new ArrayList();
		coll.add("aaa");
		coll.add("bbb");
		coll.add("ccc");
		coll.add("ddd");
		System.out.println(coll);
		Iterator it = coll.iterator();
		while (it.hasNext()) {
			it.next();
			it.remove();
		}
		System.out.println(coll);
	}
}
```

==需要注意的细节：==

细节一：

如果迭代器的指针已经指向了集合的末尾，那么如果再调用next()会返回NoSuchElementException异常。

细节二：

如果调用remove之前没有调用next是不合法的，会抛出IllegalStateException。

### 5.5 迭代器的原理
我们在ArrayList下找到了iterator实现的身影：它是在ArrayList**以内部类的方式**实现的！并且，从源码可知：**Iterator实际上就是在遍历集合**
![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ArrayList%E4%B8%AD%E7%9A%84Iterator.PNG)

所以说：我们**遍历集合(Collection)的元素都可以使用Iterator**，至于它的具体实现是以内部类的方式实现的！

注意：
1. 在对集合进行迭代过程中，不允许出现迭代器以外的对元素的操作，因为这样会产生安全隐患，java会抛出异常并发修改异常（ConcurrentModificationException），普通迭代器只支持在迭代过程中的删除动作。
2. ConcurrentModificationException:当一个集合在循环中即使用引用变量操作集合又使用迭代器操作集合对象， 会抛出该异常。

例如：

```java
public class Demo1 {
	public static void main(String[] args) {
		Collection coll = new ArrayList();
		coll.add("aaa");
		coll.add("bbb");
		coll.add("ccc");
		coll.add("ddd");
		System.out.println(coll);
		Iterator it = coll.iterator();
		while (it.hasNext()) {
			it.next();
			it.remove();
			coll.add("abc"); // 出现了迭代器以外的对元素的操作
		}
		System.out.println(coll);
	}
}
```

### 5.6 List特有的迭代器ListIterator
如果是List集合，想要在迭代中操作元素可以使用List集合的特有迭代器ListIterator，该迭代器支持在迭代过程中，添加元素和修改元素。

![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/Java%E9%9B%86%E5%90%88/ListIterator.PNG)

倒序遍历
```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
        // 获取List专属的迭代器
		ListIterator lit = list.listIterator();
		while (lit.hasNext()) {
			String next = (String) lit.next();
			System.out.println(next);
		}
		System.out.println("***************");
		while (lit.hasPrevious()) {
			String next = (String) lit.previous();
			System.out.println(next);
		}
 
	}
}
```

Set方法：用指定元素替换 next 或 previous 返回的最后一个元素
```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		ListIterator lit = list.listIterator();
		lit.next(); // 计算机网络
		lit.next(); // 现代操作系统
		System.out.println(lit.next()); // java编程思想
		//用指定元素替换 next 或 previous 返回的最后一个元素
		lit.set("平凡的世界");// 将java编程思想替换为平凡的世界
		System.out.println(list);
 
	}
}
```

add方法将指定的元素插入列表，该元素直接插入到 next 返回的元素的后面
```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		ListIterator lit = list.listIterator();
		lit.next(); // 计算机网络
		lit.next(); // 现代操作系统
		System.out.println(lit.next()); // java编程思想
		// 将指定的元素插入列表，该元素直接插入到 next 返回的元素的后
		lit.add("平凡的世界");// 在java编程思想后添加平凡的世界
		System.out.println(list);
 
	}
}
```