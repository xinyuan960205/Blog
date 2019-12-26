# AQS

**Lock的子类实现都是基于AQS的**。

# 一、AQS是什么？

首先我们来普及一下juc是什么：**juc其实就是包的缩写(java.util.concurrnt)**

- 不要被人家唬到了，以为juc是什么一个牛逼的东西。其实指的是包而已~

我们可以发现lock包下有三个抽象的类：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/lock%E5%8C%85%E4%B8%8B%E7%9A%84%E4%B8%89%E4%B8%AA%E6%8A%BD%E8%B1%A1%E7%B1%BB.PNG)

- AbstractOwnableSynchronizer
- AbstractQueuedLongSynchronizer
- AbstractQueuedSynchronizer

通常地：**AbstractQueuedSynchronizer简称为AQS**

我们Lock之类的两个常见的锁都是基于它来实现的：











