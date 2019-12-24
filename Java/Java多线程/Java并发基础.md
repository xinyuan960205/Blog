# Java并发基础

## 一、什么是并发编程

### 1.1 并发编程的目的 

> 更加充分利用计算机资源，加快程序响应速度，简化异步事件的处理

### 1.2 什么时候时候并发编程

> 任务会阻塞线程，导致之后的代码不能执行：例如边从文件读取边进行大量计算
>  任务执行时间长，可划分为分工明确的子任务：例如分段下载
> 任务间断性执行：日志打印
>  任务本身需要协作执行：例如生产者消费者问题

## 二、并发与并行

**并行：**

- 并行性是指**同一时刻内**发生两个或多个事件。
- 并行是在**不同**实体上的多个事件

**并发：**

- 并发性是指**同一时间间隔内**发生两个或多个事件。
- 并发是在**同一实体**上的多个事件

由此可见：并行是针对进程的，**并发是针对线程的**。

## 三、上下文切换

CPU为线程分配时间片，不停切换线程执行，在切换前会保存上一个任务的状态，以便切换回该任务时可以再次加载其状态。

上下文切换为什么有较大性能开销：

> - CPU从用户态要切换到内核态 
> - 要保存切换前线程的执行指针到内存块，再切换到新线程，在新线程执行完后要从内存块读回中断线程的指针继续完成执行 

 如何减少上下文切换的开销?

> - 无锁并发编程，多线程竞争锁时会引起上下文切换，所以多线程处理数据时，可以用一些办法避免使用锁，如将数据的ID按照Hash算法取模分段，不同的线程处理不同的数据 
> - CAS算法 Java的Atomic包使用CAS算法更新数据，而不需要加锁 
> - 是用最少的线程 避免创建不需要的线程，如任务状态很少但创建多线程处理，会造成大量线程处于等待状态 
> - 协程 在单线程中实现多任务调度，并在单线程里维持多任务切换。（Java很少用）

## 四、死锁

线程A获得了资源1，需要获得资源2，而线程B获得了资源2，需要获得资源1，两个线程都不释放资源又在等待对方释放资源，此时就发生了死锁。

```java
public class DeadLock {
    private static final Object resource1=new Object();
    private static final Object resource2=new Object();
 
    public static void main(String[] args) {
        Thread threadA=new Thread(()->{
            synchronized (resource1){
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (resource2){
                    System.out.println("...");
                }
            }
        });
        Thread threadB=new Thread(()->{
            synchronized (resource2){
                synchronized (resource1){
                    System.out.println("...");
                }
            }
        });
        threadA.start();
        threadB.start();
    }
}
```

运行程序时发生了死锁，我们在控制台输入：`jps` 可以找到发生死锁的java进程

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%AD%BB%E9%94%81%E6%9F%A5%E8%AF%A2jps.PNG)
然后输入`jstack 157248`可以查看到具体的死锁信息
![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E6%AD%BB%E9%94%81%E6%97%B6%E7%9A%84%E8%BF%90%E8%A1%8C%E7%8A%B6%E5%86%B5.PNG)
可以看到线程1在等待线程0的资源，而线程0也在等待线程1的资源，互不相让所以死锁了。

## 线程安全

死锁容易通过jdk的工具定位发现问题，如上述的jps，jstack，但线程安全问题运行时并无明显特征，然而结果可能不是你的预期结果。

```java
public class UnsafeThread {
    private static int num;
 
    public static void inCreate(){
        num++;
    }
 
    public static void main(String[] args) throws InterruptedException{
        //创建10000个线程，执行对num的加1操作
        for(int i=0;i<10000;i++){
            new Thread(UnsafeThread::inCreate).start();
        }
        Thread.sleep(1000);//保证线程执行完毕
        System.out.println(num);//结果可能不为10000
    }
}
```

在我们的预期中，创建10000个线程，每个线程对静态变量num执行一次++操作，结果应为10000，但输出的结果很可能不是10000，这就说明存在线程安全问题。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E9%97%AE%E9%A2%98%E5%9B%BE%E7%A4%BA.png)

有可能线程1读取了num，此时num为0，然后对它加1，由于不是同步的，在加1之前，线程2又读取到了num，由于线程1还未完成加1操作，所以线程2读取到的值也是0，然后各自完成加1操作，结果num还是为1，正常的预期结果应该是2，由此一来最终结果就未必是10000了。  



## 参考

- https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484186&idx=1&sn=2a7b937e6d3b1623aceac199d3e402f9&chksm=ebd7421bdca0cb0d6206db8c7f063c884c3f0b285975c8e896fde424660b4ccb88da1549f32c&scene=21###wechat_redirect
- https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/JavaConcurrencyBasicsCommonInterviewQuestionsSummary.md
- https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20并发.md#二基础线程机制





