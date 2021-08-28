# JAVA 多线程

## 线程的状态

1. 创建线程
2. 就绪状态
3. 运行状态
4. 阻塞状态
5. 等待状态
6. 锁池状态
7. 死亡



## 进程和线程

进程就好比火车，线程就好比车厢。

其中进程间的通信方式有：管道，信号量，共享内存，消息队列。

其中线程间的通信方式有：共享变量，共享队列。



## synchronized 和 volatile 区别

1. volatile 只能用于变量，synchronized 可用于变量，方法，类，代码块。
2. volatile 不能保证原子性。
3. volatile 不会造成阻塞，synchronized 可能会造成阻塞。

**Synchronized 去修饰一个方法的时候。它是给类加锁？还是给对象加锁？**

1. 如果方法是静态的，那么加的是类锁。
2. 如果方法是非静态的，加的是对象锁。
3. 如果修饰的是代码块，加的也是对象锁。



## wait 和 sleep 

1. wait 会释放对象锁。
2. sleep 不会释放对象锁。



## 创建多线程的方式

1. 继承 Thread 类，然后重写 run() 方法。
2. 实现 Runnable 接口，然后重写 run() 方法。
3. 实现 Callable 接口，然后重写 call() 方法。



## JAVA 创建线程池的方法

**创建线程池的方法总共分为两大类：**

1. 通过 ThreadPoolExecutor 创建线程池。
2. 通过 Executors 创建线程池。



**创建线程池的方法：**

1. Executors.newFixedThreadPool：创建一个固定大小的线程池，可控制并发的线程数，超出的线程会在队列中等待；
2. Executors.newCachedThreadPool：创建一个可缓存的线程池，若线程数超过处理所需，缓存一段时间后会回收，若线程数不够，则新建线程；
3. Executors.newSingleThreadExecutor：创建单个线程数的线程池，它可以保证先进先出的执行顺序；
4. Executors.newScheduledThreadPool：创建一个可以执行延迟任务的线程池；
5. Executors.newSingleThreadScheduledExecutor：创建一个单线程的可以执行延迟任务的线程池；
6. Executors.newWorkStealingPool：创建一个抢占式执行的线程池（任务执行顺序不确定）【JDK 1.8 添加】。
7. ThreadPoolExecutor：最原始的创建线程池的方式，它包含了 7 个参数可供设置，后面会详细讲。



**为什么不建议使用 Executors 创建线程池？**

1. FixedThreadPool 和 SingleThreadPool：允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。（底层使用了队列 LinkedBlockingQueue ，这是一个链表实现的有界阻塞队列，容量可以进行设置，不设置的话将是一个无边界的阻塞队列，无边界的队列，任务会一直写入，很容易填满内存）
2. CachedThreadPool：允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。（创建线程池的数量是无限大，无线创建线程，很容易把内存填满）



**ThreadPoolExecutor**

```java
public ThreadPoolExecutor(int corePoolSize,//10,核心线程数，线程池中始终存活的线程数。
                              int maximumPoolSize,//30,最大线程数，线程池中允许的最大线程数，当线程池的任务队列满了之后可以创建的最大线程数。
                              long keepAliveTime,//最大线程数可以存活的时间，当线程中没有任务执行时，最大线程就会销毁一部分，最终保持核心线程数量的线程。
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,//1000,一个阻塞队列，用来存储线程池等待执行的任务，均为线程安全。
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

1. 当接收到 30 个比较耗时的任务时，10 个核心线程数（corePoolSize）都在工作，剩余的 20 个去队列（workQueue）里排队。
2. 这个线程池最多接收的任务：maximumPoolSize + workQueue。



**工作机制：**

1. 当线程数小于核心线程数（corePoolSize）时，创建线程。
2. 当线程数大于等于核心线程数（corePoolSize），且任务队列（workQueue）未满时，将任务放入任务队列（workQueue）。
3. 当线程数大于等于核心线程数（corePoolSize），且任务队列（workQueue）已满：若线程数小于最大线程数（maximumPoolSize ），创建线程；若线程数等于最大线程数（maximumPoolSize ），抛出异常，拒绝任务。



## 线程池拒绝策略

1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。（ThreadPoolExecutor 线程池默认策略）
2. ThreadPoolExecutor.DiscardPolicy：丢弃任务，但是不抛出异常。
3. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务。
4. ThreadPoolExecutor.CallerRunsPolicy：由调用线程（提交任务的线程）处理该任务。



## Java内存CPU占用过高排查

1. ps -ef | grep tomcat ------> 拿到 Tomcat 进程的 pid 。
2. jstack -l 进程pid >> jstack.log ------> 打印并保存该进程中堆栈的使用信息日志 。
3. top -Hp pid ------> 展示进程中所有线程的 cpu 占用情况 。
4. printf %x 线程pid ------> 该线程对应的 16 进制 。
5. vim jstack.log ------> 编辑查找 3 中打印的 16 进制值 。
6. 分析并定位到问题代码 