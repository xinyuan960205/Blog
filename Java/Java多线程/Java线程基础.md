# Java线程基础

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

## 二、Java实现多线程

Java实现多线程是使用Thread这个类的，我们来看看**Thread类的顶部注释**：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/Thread%E9%A1%B6%E9%83%A8%E6%B3%A8%E9%87%8A.jpg)

通过上面的顶部注释我们就可以发现，创建多线程有**两种**方法：

- 继承Thread，重写run方法
- 实现Runnable接口，重写run方法

那么，既然有两种方式实现多线程，我们**使用哪一种**？？？

**一般我们使用实现Runnable接口**

- **可以避免java中的单继承的限制**
- 应该将并发**运行任务和运行机制解耦**，因此我们选择实现Runnable接口这种方式！

### 2.1 继承Thread，重写run方法

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

### 2.2 实现Runnable接口，重写run方法

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

//补充：也可使用线程池 将实现Runnable接口的类的实例作为参数传入executorService
public class MyThread implements Runnable  {
    @Override
    public void run() { ...}
 
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        executorService.execute(new MyThread());
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

### 2.3 Java实现多线程需要注意的细节

不要将`run()`和`start()`搞混了~

**run()和start()方法区别：**

- `run()`:仅仅是**封装被线程执行的代码**，直接调用是普通方法
- `start()`:首先**启动了线程**，然后再**由jvm去调用该线程的run()方法。**



面试问题：**为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？**

这是另一个非常经典的 java 多线程面试问题，而且在面试中会经常被问到。很简单，但是很多人都会答不上来！

new 一个 Thread，线程进入了新建状态;调用 start() 方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。 而直接执行 run() 方法，会把 run 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 start 方法方可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法调用，还是在主线程里执行。**

## 三、线程的生命周期

Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态（图源《Java 并发编程艺术》4.1.4 节）。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/java%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%85%AD%E7%A7%8D%E4%B8%8D%E5%90%8C%E7%8A%B6%E6%80%81.png)

线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。Java 线程状态变迁如下图所示（图源《Java 并发编程艺术》4.1.4 节）：

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/java%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E5%8C%96.png)



由上图可以看出：线程创建之后它将处于 **NEW（新建）** 状态，调用 `start()` 方法后开始运行，线程这时候处于 **READY（可运行）** 状态。可运行状态的线程获得了 CPU 时间片（timeslice）后就处于 **RUNNING（运行）** 状态。

> 操作系统隐藏 Java 虚拟机（JVM）中的 RUNNABLE 和 RUNNING 状态，它只能看到 RUNNABLE 状态（图源：[HowToDoInJava](https://howtodoinjava.com/)：[Java Thread Life Cycle and Thread States](https://howtodoinjava.com/java/multi-threading/java-thread-life-cycle-and-thread-states/)），所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/JVM%E4%B8%AD%E7%9A%84ready%E5%92%8Crunning%E7%8A%B6%E6%80%81.png)

当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）** 状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而 **TIME_WAITING(超时等待)** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将 Java 线程置于 TIMED WAITING 状态。当超时时间到达后 Java 线程将会返回到 RUNNABLE 状态。当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到 **BLOCKED（阻塞）** 状态。线程在执行 Runnable 的`run()`方法之后将会进入到 **TERMINATED（终止）** 状态。

### 3.1 sleep方法

调用sleep方法会进入计时等待状态，等时间到了，**进入的是就绪状态而并非是运行状态**！

### 3.2 yield方法

调用yield方法会先**让别的线程执行**，但是**不确保真正让出**

- 意思是：**我有空，可以的话，让你们先执行**

### 3.3 join方法

调用join方法，会等待**该线程**执行**完毕后才执行别的线程**~

### 3.4 线程的挂起和恢复

- 什么是挂起线程 

> 实质上是指使线程进入非RUNNABLE状态，该状态下CPU不会给线程分配时间片，进入该状态可用来暂停一个线程的运行。
> 线程挂起后，可通过重新唤醒线程使其恢复运行。

- 为什么要挂起线程 

> CPU分配时间片非常短，资源很珍贵，挂起无用线程可避免资源的浪费。

- 如何挂起线程 

> 被废弃的方法：
> **suspend()** 该方法不会释放线程所占资源，如果使用该方法将某个线程挂起，则可能会使其他资源的线程死锁 
>
> **resume()**  方法本身无问题，但不能独立于suspend()方法存在
> 可以使用的方法：
>  **wait()** 暂停执行，放弃已获得的锁，进入等待状态
> **notift()/notifyAll()** 随即唤醒一个等待锁的线程/唤醒所有等待锁的线程，自行抢占CPU资源

```java
public class WaitTest implements Runnable  {
    private static final Object lock=new Object();
    @Override
    public void run() {
        synchronized (lock){
            System.out.println("开始，线程准备挂起");
            try {
                lock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程恢复运行，结束");
        }
    }
 
    public static void main(String[] args) throws InterruptedException {
        Thread waitThread = new Thread(new WaitTest());
        waitThread.start();
        Thread.sleep(100);//保证线程挂起发生于唤醒前
        synchronized (lock) {//挂起和唤醒需要持有相同的锁
            System.out.println("准备唤醒挂起线程");
            lock.notify();
        }
    }
}
 
输出结果：
开始，线程准备挂起
准备唤醒挂起线程
线程恢复运行，结束
```

- 什么时候适合挂起线程
  等待某些未就绪的资源，直到被唤醒

### 3.5 线程的中断操作

 废弃方法：

stop() 因为一旦调用线程就立刻停止，可能引发线程安全问题。

可使用方法：

interrupt() 向线程发送中断请求，将中断状态置为true，如果线程仍在运行并不会中断

interrupted() 静态方法，检测当前线程是否被中断，如中断状态为true则中断

#### 3.5.1 InterruptedException

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
    
```

#### 3.5.2 interrupted()

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

```java
public class InterruptTest implements Runnable {
 
    @Override
    public void run() {
        while (!Thread.interrupted()){//若为while(true)则interrupt不会中断程序
            System.out.println("123");
        }
    }
 
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new InterruptTest());
        thread.start();
        Thread.sleep(100);//保证执行先于中断
        thread.interrupt();//程序在此处结束
    }
}
```

自行定义一个标志，需要用volatile修饰

```java
public class InterruptTest implements Runnable {
    private static volatile boolean flag=true;//不加volatile可能出错，保证修改可见性
 
    @Override
    public void run() {
        while (flag){
            System.out.println("123");
        }
    }
 
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new InterruptTest());
        thread.start();
        Thread.sleep(100);
        flag=false;//程序中断
    }
}
```

#### 3.5.3 Executor 的中断操作

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
```

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

```java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

## 四、线程的优先级

当大量线程在等待运行时，CPU优先将时间片分配给优先级高的线程，这不代表优先级低的线程不会运行，只是它被运行的机会会小一点。
线程优先级可以为1-10的任意数值，Thread中定义了三个线程优先级，MIN_PRIORITY(1)、NORM_PRIORITY(5)、MAX_PRIORITY(10)，一般情况推荐使用这三个常量，不要自行设置。
使用    `setPriority()`设置优先级。
不同平台对优先级的支持不同，尽量不要依赖优先级。  

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E7%BA%BF%E7%A8%8B%E4%BC%98%E5%85%88%E7%BA%A7%E7%9A%84%E5%AE%9E%E7%8E%B0.webp)

`setPriority0`是一个本地(navite)的方法：

```java
 private native void setPriority0(int newPriority);
```

## 五、守护线程

- 线程分类：
  1. 用户线程 用户线程是独立存在的，不会因为其他用户线程退出而退出
  2. 守护线程 依赖于用户线程，只要有活着的用户线程，守护线程就活着。当JVM中最后一个非守护线程结束时，就随JVM一起退出。 

- 用处：
  JVM垃圾清理线程 

- 建议：
  尽量不使用守护线程，因其不可控，不要在守护线程中读写文件，执行计算逻辑 

- 设置守护线程
  `setDaemon(true)`，需要在调用start()方法前设置。