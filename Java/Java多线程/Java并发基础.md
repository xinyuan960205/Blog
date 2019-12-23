# Java并发基础

## 一、线程和进程

### 1.1 进程

> 进程是**程序的一次执行**，进程是一个程序及其数据在处理机上顺序执行时所发生的**活动**，进程是具有独立功能的程序在一个数据集合上运行的过程，它是系统进行资源**分配和调度的一个独立单位**

- **进程是系统进行资源分配和调度的独立单位。每一个进程都有它自己的内存空间和系统资源**

在 Java 中，当我们启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称主线程。

如下图所示，在 windows 中通过查看任务管理器的方式，我们就可以清楚看到 window 当前运行的进程（.exe 文件的运行）。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545382542462539422545372541382538422545372541342542412545342542452538422545352539422542452545372538392538372d57696e646f77732e706e67.png)

### 1.2 线程

线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

Java 程序天生就是多线程程序，我们可以通过 JMX 来看一下一个普通的 Java 程序有哪些线程，代码如下。

```java
public class MultiThread {
	public static void main(String[] args) {
		// 获取 Java 线程管理 MXBean
		ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
		// 不需要获取同步的 monitor 和 synchronizer 信息，仅获取线程和线程堆栈信息
		ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
		// 遍历线程信息，仅打印线程 ID 和线程名称信息
		for (ThreadInfo threadInfo : threadInfos) {
			System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
		}
	}
}
```

上述程序输出如下（输出内容可能不同，不用太纠结下面每个线程的作用，只用知道 main 线程执行 main 方法即可）：

```
[5] Attach Listener //添加事件
[4] Signal Dispatcher // 分发处理给 JVM 信号的线程
[3] Finalizer //调用对象 finalize 方法的线程
[2] Reference Handler //清除 reference 线程
[1] main //main 线程,程序入口
```

从上面的输出内容可以看出：**一个 Java 程序的运行是 main 线程和多个其他线程同时运行**。

### 1.3 为什么要有线程

为使程序能并发执行，系统必须进行以下的一系列操作：

- (1)**创建进程**，系统在创建一个进程时，必须为它分配其所必需的、除处理机以外的所有资源，如内存空间、I/O设备，以及建立相应的PCB；
- (2)**撤消进程**，系统在撤消进程时，又必须先对其所占有的资源执行回收操作，然后再撤消PCB；
- (3)**进程切换**，对进程进行上下文切换时，需要保留当前进程的CPU环境，设置新选中进程的CPU环境，因而须花费不少的处理机时间。

可以看到进程实现多处理机环境下的进程调度，分派，切换时，**都需要花费较大的时间和空间开销**

引入线程主要是**为了提高系统的执行效率，减少处理机的空转时间和调度切换的时间，以及便于系统管理。**使OS具有更好的并发性

### 1.4 从 JVM 角度说进程和线程之间的关系

#### 1.4.1 图解进程和线程的关系

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/%E5%9B%BE%E7%9A%84%E8%BF%9B%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E5%85%B3%E7%B3%BB.png)

从上图可以看出：一个进程中可以有多个线程，多个线程共享进程的**堆**和**方法区 (JDK1.8 之后的元空间)**资源，但是每个线程有自己的**程序计数器**、**虚拟机栈** 和 **本地方法栈**。

**总结：** 线程 是 进程 划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反

！！！后面的都是补充

#### 1.4.2 程序计数器为什么是私有的?

程序计数器主要有下面两个作用：

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

需要注意的是，如果执行的是 native 方法，那么程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址。

所以，程序计数器私有主要是为了**线程切换后能恢复到正确的执行位置**。

#### 1.4.3 虚拟机栈和本地方法栈为什么是私有的?

- **虚拟机栈：** 每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。
- **本地方法栈：** 和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

所以，为了**保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的。

#### 1.4.4 一句话简单了解堆和方法区

堆和方法区是所有线程共享的资源，其中堆是进程中最大的一块内存，主要用于存放新创建的对象 (所有对象都在这里分配内存)，方法区主要用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

## 二、并发与并行

**并行：**

- 并行性是指**同一时刻内**发生两个或多个事件。
- 并行是在**不同**实体上的多个事件

**并发：**

- 并发性是指**同一时间间隔内**发生两个或多个事件。
- 并发是在**同一实体**上的多个事件

由此可见：并行是针对进程的，**并发是针对线程的**。

## 三、Java实现多线程

Java实现多线程是使用Thread这个类的，我们来看看**Thread类的顶部注释**：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/Thread%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.jpg)

通过上面的顶部注释我们就可以发现，创建多线程有**两种**方法：

- 继承Thread，重写run方法
- 实现Runnable接口，重写run方法

那么，既然有两种方式实现多线程，我们**使用哪一种**？？？

**一般我们使用实现Runnable接口**

- **可以避免java中的单继承的限制**
- 应该将并发**运行任务和运行机制解耦**，因此我们选择实现Runnable接口这种方式！

### 3.1 继承Thread，重写run方法

**创建一个类，继承Thread，重写run方法**

```Java
public class MyThread extends Thread {

    @Override
    public void run() {
        for (int x = 0; x < 200; x++) {
            System.out.println(x);
        }
    }

}
```

我们调用一下测试看看：

```java
public class MyThreadDemo {
    public static void main(String[] args) {
        // 创建两个线程对象
        MyThread my1 = new MyThread();
        MyThread my2 = new MyThread();

        my1.start();
        my2.start();
    }
}
```

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib0bjBejhzqrhcUsVWiaON4uVAecqQwvu8wkUNSZjcLt1mYhfP8rR7eCIMcKTDzfcxq291QMEVWcibFQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 3.2 实现Runnable接口，重写run方法

**实现Runnable接口，重写run方法**

```java
public class MyRunnable implements Runnable {

    @Override
    public void run() {
        for (int x = 0; x < 100; x++) {
            System.out.println(x);
        }
    }

}
```

我们调用一下测试看看：

```java
public class MyRunnableDemo {
    public static void main(String[] args) {
        // 创建MyRunnable类的对象
        MyRunnable my = new MyRunnable();

        Thread t1 = new Thread(my);
        Thread t2 = new Thread(my);

        t1.start();
        t2.start();
    }
}
```

结果还是跟上面是**一样**的，这里我就不贴图了~~~

### 3.3 Java实现多线程需要注意的细节

不要将`run()`和`start()`搞混了~

**run()和start()方法区别：**

- `run()`:仅仅是**封装被线程执行的代码**，直接调用是普通方法
- `start()`:首先**启动了线程**，然后再**由jvm去调用该线程的run()方法。**



面试问题：**为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？**

这是另一个非常经典的 java 多线程面试问题，而且在面试中会经常被问到。很简单，但是很多人都会答不上来！

new 一个 Thread，线程进入了新建状态;调用 start() 方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。 而直接执行 run() 方法，会把 run 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 start 方法方可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法调用，还是在主线程里执行。**





## 参考

- https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484186&idx=1&sn=2a7b937e6d3b1623aceac199d3e402f9&chksm=ebd7421bdca0cb0d6206db8c7f063c884c3f0b285975c8e896fde424660b4ccb88da1549f32c&scene=21###wechat_redirect
- https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/JavaConcurrencyBasicsCommonInterviewQuestionsSummary.md
- https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20并发.md#二基础线程机制





