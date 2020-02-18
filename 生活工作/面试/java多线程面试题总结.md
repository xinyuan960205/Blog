## java多线程基础

- 进程线程区别，线程安全和非线程安全区别
- 线程状态，start,run,wait,notify,yiled,sleep,join等方法的作用以及区别
- wait,notify阻塞唤醒确切过程？在哪阻塞，在哪唤醒？为什么要出现在同步代码块中，为什么要处于while循环中？
- 线程中断，守护线程
- Thread，Runnable，Callable，Future，ThreadLocal 等，会用，知道区别；



## java并发问题







## java锁机制

- Java乐观锁机制，CAS思想？缺点？是否原子性？如何保证？，知道 Java 中的 Unsafe 类（知道用这个东西就行了，直接操作内存的，基本用不到）；
- synchronized使用方法？底层实现？首先知道它是干啥的，其次了解咋实现的，我认为至少要讲到 monitor 面试官才会满意，其间可能牵扯到堆中对象头结构（这里还可能牵扯到数组的 length 属性咋来的）
- Lock，主要是ReenTrantLock使用方法？底层实现？和synchronized区别？
- 公平锁和非公平锁区别？为什么公平锁效率低？
- 锁优化。自旋锁、自适应自旋锁、锁消除（这里还可能牵扯到对象逃逸等）、锁粗化、偏向锁、轻量级锁、重量级锁解释，对象头的 Mark Word 复用等
- 了解 AQS 的基本工作原理，包括同步队列和 Condition 的等待队列等（和 monitor 的队列很像），了解它的 states 状态是用 CAS 算法更新的





## 线程池

- 线程池体系，ExecutorService，ScheduledExecutorService 等接口，ThreadPoolExecutor，ForkJoinPool 等实现类，Executors 静态工厂类，线程池的核心参数，几种不同类型的工厂线程池等，会用，了解一下；
- 线程池构造函数7大参数，线程处理任务过程，线程拒绝策略
- Execuors类实现的几种线程池类型，阿里为啥不让用？
- 线程池大小如何设置？





## 并发集合







## 并发理论

- Java内存模型
- 了解可见性问题（由此引出 volatile 关键字），知道 volatile 能解决可见性（区别于原子性）和重排序问题，由此引出有序性的先行发生原则（happens-before）。
- volatile作用？底层实现？禁止重排序的场景？单例模式中volatile的作用？







## 并发工具类

- 基于AQS实现的lock, CountDownLatch、CyclicBarrier、Semaphore介绍



## 场景

- 手写简单的线程池，体现线程复用
- 手写消费者生产者模式
- 手写阻塞队列
- 手写多线程交替打印ABC









