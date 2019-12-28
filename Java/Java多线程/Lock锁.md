# Lock锁

## 一、Lock锁介绍

Lock显式锁是JDK1.5之后才有的，之前我们都是使用Synchronized锁来使线程安全的~

Lock显式锁是一个接口，我们来看看：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/Lock%E9%94%81.webp)

随便翻译一下他的顶部注释，看看是干嘛用的：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/Lock%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.webp)

可以**简单概括**一下：

- Lock方式来获取锁**支持中断、超时不获取、是非阻塞的**
- **提高了语义化**，哪里加锁，哪里解锁都得写出来
- **Lock显式锁可以给我们带来很好的灵活性，但同时我们必须手动释放锁**
- 支持Condition条件对象
- **允许多个读线程同时访问共享资源**

## 二、synchronized锁和Lock锁使用哪个

前面说了，Lock显式锁给我们的程序带来了很多的灵活性，很多特性都是Synchronized锁没有的。那Synchronized锁有没有存在的必要？？

必须是有的！！Lock锁在刚出来的时候很多性能方面都比Synchronized锁要好，但是从JDK1.6开始Synchronized锁就做了各种的优化(毕竟亲儿子，牛逼)

- 优化操作：适应自旋锁，锁消除，锁粗化，轻量级锁，偏向锁。

所以，到现在Lock锁和Synchronized锁的性能其实**差别不是很大**！而Synchronized锁用起来又特别简单。**Lock锁还得顾忌到它的特性，要手动释放锁才行**(如果忘了释放，这就是一个隐患)

所以说，我们**绝大部分时候还是会使用Synchronized锁**，用到了Lock锁提及的特性，带来的灵活性才会考虑使用Lock显式锁~

## 三、公平锁

公平锁理解起来非常简单：

- 线程将按照它们**发出请求的顺序来获取锁**

非公平锁就是：

- 线程发出请求的时可以**“插队”**获取锁

Lock和synchronize都是**默认使用非公平锁的**。如果不是必要的情况下，不要使用公平锁

- 公平锁会来带一些性能的消耗的

## 四、ReentrantLock锁

首先我们来看看ReentrantLock锁的**顶部注释**，来看看他的相关特性呗：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/ReentrantLock%E9%94%81%E7%9A%84%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.webp)

来总结一下要点吧：

- 比synchronized更有伸缩性(灵活)
- 支持公平锁(是相对公平的)
- 使用时最标准用法是在try之前调用lock方法，在finally代码块释放锁

```java
class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...

    public void m() { 
        lock.lock();  // block until condition holds
        try {
            // ... method body
        } finally {
            lock.unlock()
        }
    }
}
```

### 内部类

首先我们可以看到有三个内部类：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/RTL%E5%86%85%E9%83%A8%E7%B1%BB.webp)

这些内部类都是AQS的子类，这就印证了我们之前所说的：**AQS是ReentrantLock的基础，AQS是构建锁、同步器的框架**

- 可以很清晰的看到，我们的ReentrantLock锁是支持公平锁和非公平锁的~

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/ReentrantLock%E7%BB%A7%E6%89%BF%E5%9B%BE.webp)

### 构造方法

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/ReentrantLock%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.webp)

### 非公平lock方法

尝试获取锁，获取失败的话就调用AQS的`acquire(1)`方法

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%B0%83%E7%94%A8AQS%E7%9A%84acquire(1)%E6%96%B9%E6%B3%95.webp)

`acquire(1)`方法我们在AQS时简单看过，其中`tryAcquire()`是子类来实现的

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/tryAcquire()%E5%AD%90%E7%B1%BB%E5%AE%9E%E7%8E%B0.webp)

我们去看看`tryAcquire()`：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/tryAcquire().webp)

### 公平lock方法

公平的lock方法其实就**多了一个状态条件**：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%85%AC%E5%B9%B3%E7%9A%84lock%E6%96%B9%E6%B3%95.webp)

这个方法主要是判断**当前线程是否位于CLH同步队列中的第一个。如果是则返回flase，否则返回true**。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/CLH%E5%90%8C%E6%AD%A5%E9%98%9F%E5%88%97.webp)

### unlock方法

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/unlock%E6%96%B9%E6%B3%95.webp)

unlock方法也是在AQS中定义的：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/unlock%E6%96%B9%E6%B3%95%E4%B9%9F%E6%98%AF%E5%9C%A8AQS%E4%B8%AD%E5%AE%9A%E4%B9%89.webp)

去看看`tryRelease(arg)`是怎么实现的：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/tryRelease(arg)%E5%AE%9E%E7%8E%B0.webp)

## 五、ReentrantReadWriteLock

我们知道synchronized内置锁和ReentrantLock都是**互斥锁**(一次只能有一个线程进入到临界区(被锁定的区域))

而ReentrantReadWriteLock是一个**读写锁**：

- 在**读**取数据的时候，可以**多个线程同时进入到到临界区**(被锁定的区域)
- 在**写**数据的时候，无论是读线程还是写线程都是**互斥**的

一般来说：我们大多数都是读取数据得多，修改数据得少。所以这个读写锁在这种场景下就很有用了！

读写锁有一个接口ReadWriteLock，定义的方法就两个：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E6%8E%A5%E5%8F%A3ReadWriteLock%E5%AE%9A%E4%B9%89.webp)

我们还是来看看顶部注释说得啥吧：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.webp)

其实大概也是说明了：**在读的时候可以共享，在写的时候是互斥的**

接下来我们还是来看看对应的实现类吧：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/ReentrantReadWriteLock%E5%AE%9E%E7%8E%B0%E7%B1%BB.webp)

按照惯例也简单看看它的顶部注释：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%AE%9E%E7%8E%B0%E7%B1%BB%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.webp)

于是我们可以总结出读写锁的一些要点了：

- 读锁不支持条件对象，写锁支持条件对象
- 读锁不能升级为写锁，写锁可以降级为读锁
- 读写锁也有公平和非公平模式
- **读锁支持多个读线程进入临界区，写锁是互斥的**

### ReentrantReadWriteLock内部类

ReentrantReadWriteLock比ReentrantLock锁**多了两个内部类(都是Lock实现)来维护读锁和写锁**，但是**主体还是使用Syn**：

- WriteLock
- ReadLock

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%AF%BB%E9%94%81%E5%92%8C%E5%86%99%E9%94%81%E7%9A%84%E7%BB%B4%E6%8A%A4.webp)

### 读锁和写锁的状态表示

在ReentrantLock锁上使用的是state来表示同步状态(也可以表示重入的次数)，而在ReentrantReadWriteLock是这样代表读写状态的：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%AF%BB%E9%94%81%E5%92%8C%E5%86%99%E9%94%81%E7%9A%84%E7%8A%B6%E6%80%81%E8%A1%A8%E7%A4%BA.webp)

### 写锁的获取

主要还是调用syn的`acquire(1)`：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%86%99%E9%94%81%E7%9A%84%E8%8E%B7%E5%8F%96.webp)

进去看看实现：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%BF%9B%E5%8E%BB%E7%9C%8B%E7%9C%8B%E5%86%99%E9%94%81%E7%9A%84%E5%AE%9E%E7%8E%B0.webp)

### 读锁获取

读锁的获取调用的是`acquireShared(int arg)`方法：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%AF%BB%E9%94%81%E8%8E%B7%E5%8F%96.webp)

内部调用的是：`doAcquireShared(arg);`方法(实现也是在Syn的)，我们来看看：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%86%85%E9%83%A8%E8%B0%83%E7%94%A8%E7%9A%84doAcquireShared(arg).webp)











