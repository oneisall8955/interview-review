# 线程池相关
`Executor`框架主要三个部分组成：
+ 任务（Runnable/Callable）

    执行任务需要实现的`Runnable`接口 或`Callable`接口。
    `Runnable`接口或`Callable`接口实现类都可以被`ThreadPoolExecutor`或`ScheduledThreadPoolExecutor`执行。
+ 任务的执行（Executor）

    包括任务执行机制的核心接口`Executor`，以及继承自`Executor`接口的`ExecutorService`接口。
    `ThreadPoolExecutor`和`ScheduledThreadPoolExecutor`这两个关键类实现了`ExecutorService`接口。
    
    **重点关注的是`ThreadPoolExecutor`**。
+ 异步计算的结果（Future）
    
    `Future`接口以及`Future`接口的实现类`FutureTask`类都可以代表异步计算的结果。
    
    当我们把`Runnable接口`或`Callable`接口的实现类提交
    给`ThreadPoolExecutor`或`ScheduledThreadPoolExecutor`执行。
    （调用`submit()`方法时会返回一个`FutureTask`对象）
___    

## 目录

- [任务（Runnable/Callable）和 异步计算的结果（Future](#任务（Runnable/Callable）和异步计算的结果（Future）)
    - [线程](#线程)
    - [Runnable/Callable/Future](#Runnable/Callable/Future)
- [任务的执行（Executor）](#任务的执行（Executor）)
    - [ThreadPoolExecutor](#ThreadPoolExecutor)
    - [ScheduledThreadPoolExecutor](#ScheduledThreadPoolExecutor)
    - [Executors](#Executors)
        - [Executors提供的线程池创建方法](#Executors提供的线程池创建方法)
    
___

## 任务（Runnable/Callable）和异步计算的结果（Future）
这里先简单回顾一下线程、`Runnable`和`Callable`相关内容。

### 线程
1. 生命周期：线程在生命周期内有5种状态：

    **NEW(新建)**、**RUNNABLE(就绪)**、**RUNNING(运行)**、**BLOCKED(阻塞)**、**DEAD(终止)**。
    + **NEW(新建)**：线程被创建且为启动。
    + **RUNNABLE(就绪)**：调用start()之后运行前。
    + **RUNNING(运行)**：run()正在执行。
    + **BLOCKED(阻塞)**：线程被阻塞。包括同步阻塞（锁被其他线程占用），主动阻塞（主动让出CPU执行权）、等待阻塞（执行了wait()方法）。
    + **DEAD(终止)**：run()执行结束或异常退出。

    ```
    线程状态转换关系
             ┌ ← BLOCKED ← ┐
             ↓             ↑
    NEW → RUNNABLE  ⇄  RUNNING → DEAD 
    ```
2. JVM中的线程

    多个线程共享进程的**堆**和**方法区**(JDK1.8 之后的元空间)资源，
    但是每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**。
    
    ```
    JVM中的线程内存结构
    虚拟机栈：VM Stack
    本地方法栈：Native Method Stack
    程序计数器：Program Counter Register
    +--------------------------------+   +--------------------------------+
    |             Thread             |   |             Thread             |
    +  +--------------------------+  +   +  +--------------------------+  +
    |  |         VM Stack         |  |   |  |         VM Stack         |  |
    +  +==========================+  +   +  +==========================+  +
    |  |    Native Method Stack   |  |   |  |    Native Method Stack   |  |
    +  +==========================+  +   +  +==========================+  +
    |  | Program Counter Register |  |   |  | Program Counter Register |  |
    +  +--------------------------+  +   +  +--------------------------+  +
    |                                |   |                                |
    +--------------------------------+   +--------------------------------+
    ```
    这里就说那么多了，具体等到JVM复习时再来深入了解。

3. 经典问题
    + 进程和线程的关系
    + 线程的生命周期
    + 创建线程的几种方式
    + 待补充    
### Runnable/Callable/Future
+ `Runnable`接口

    属于`java.lang`包，是线程执行类必须实现此接口，并重写`run()`方法。
    实现此方法不会返回结果或抛出检查异常。一般常用实现`Runnable`接口来创建线程类，
    可以避免单继承的局限性。

+ `Callable<V>`接口

    属于`java.util.concurrent`包，泛型接口，
    实现此接口需要重写`call()`方法，该方法会返回传入泛型类型，也可以抛出异常。
    一般是和`ExecutorService`配合使用的。

+ `Future`接口    

    属于`java.util.concurrent`包，泛型接口，
    对于具体的`Runnable`或者`Callable`任务的执行结果进行取消、查询是否完成、获取结果。
    
    ```
    public interface Future<V> {
        // 尝试取消执行此任务，在任务已经完成、任务已取消、任务不能被取消等原因返回false
        boolean cancel(boolean mayInterruptIfRunning);
        // 任务是否被取消成功
        boolean isCancelled();
        // 任务是否已经完成
        boolean isDone();
        // 获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回
        V get() throws InterruptedException, ExecutionException;
        // 获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null
        V get(long timeout, TimeUnit unit)
            throws InterruptedException, ExecutionException, TimeoutException;
    }
    ```
+ `FutureTask`
    
    `FutureTask`类实现了`RunnableFuture`接口，
    `RunnableFuture`接口继承了`Runnable`和`Future`接口，为`Future`唯一实现类。
    所以`FutureTask`又可以当做`Runnable`被线程执行，又可以作为`Future`得到`Callable`的返回值。

    + `FutureTask`有7种状态：
    
        + NEW（初始化）、COMPLETING（完成中）、NORMAL（正常结束）、EXCEPTIONAL（异常结束）、
        CANCELLED（被取消）、INTERRUPTING（正在中断）、INTERRUPTED（执行中被中断）。
    
        + 可能的状态转换：
            NEW -> COMPLETING -> NORMAL
            NEW -> COMPLETING -> EXCEPTIONAL
            NEW -> CANCELLED
            NEW -> INTERRUPTING -> INTERRUPTED
            
    + `FutureTask`有2种构造方式：
        + 传入`Callable`
        ```
        public FutureTask(Callable<V> callable) {
            if (callable == null)
                throw new NullPointerException();
            this.callable = callable;
            this.state = NEW;       // ensure visibility of callable
        }
        ```
        + 传入Runnable和result，通过Executors适配为一个Callable
        ```
        public FutureTask(Runnable runnable, V result) {
            this.callable = Executors.callable(runnable, result);
            this.state = NEW;       // ensure visibility of callable
        }    
        ```
    (暂缓)    
___ 
## 任务的执行（Executor）
### ThreadPoolExecutor
`ThreadPoolExecutor`是`Executor`框架最核心的线程池实现类。`ScheduledThreadPoolExecutor`是其子类。
继承了`AbstractExecutorService`类（实现了`ExecutorService`->`Executor`）。

+ 构造方法
    
    `ThreadPoolExecutor`提供了4个构造方法，下面这个为最基础的构造方法：
    ```
    public ThreadPoolExecutor(int corePoolSize,         // 线程池的核心线程数量
                              int maximumPoolSize,      // 线程池的最大线程数
                              long keepAliveTime,       // 当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                              TimeUnit unit,            // 时间单位
                              BlockingQueue<Runnable> workQueue,  // 任务队列，用来储存等待执行任务的队列
                              ThreadFactory threadFactory,        // 线程工厂，用来创建线程，一般默认即可
                              RejectedExecutionHandler handler    //拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                              ) {
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
    这里包括了7个参数，其中3个加粗的参数为重要参数：
    + **corePoolSize**：核心线程数线程数定义了最小可以同时运行的线程数量。
    + **maximumPoolSize**: 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
    + keepAliveTime：当线程池中的线程数量大于`corePoolSize`的时候，如果这时没有新的任务提交，
        核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了`keepAliveTime`才会被回收销毁；
    + unit：`keepAliveTime`参数的时间单位。
    + **workQueue**: 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，信任就会被存放在队列中。
    + threadFactory：`executor`创建新线程的时候会用到。
    + handler：饱和策略，拒绝策略。

+ 工作队列workQueue
    
    工作队列workQueue需要传入`BlockingQueue<Runnable>`的阻塞队列。
    
    `BlockingQueue`的实现类有`ArrayBlockingQueue`、`LinkedBlockingQueue`、
    `PriorityBlockingQueue`、`SynchronousQueue`、`DelayQueue`、`LinkedTransferQueue`。
    
    常用有`ArrayBlockingQueue`、`LinkedBlockingQueue`、`PriorityBlockingQueue`、`SynchronousQueue`用于阻塞队列。
    点击查看关于[BlockingQueue](juc.md#BlockingQueue)的介绍。
    + `ArrayBlockingQueue`：由数组实现的有界队列，遵循FIFO，长度在初始化时必须指定。
    + `LinkedBlockingQueue`（常用）：由单链表实现的边界可选队列，遵循FIFO。默认初始化`Interger.MAX_VALUE`长度的队列，可视为无界队列。传入指定长度即为有界队列。
    + `PriorityBlockingQueue`：由（二叉堆）数组实现的优先级无界队列。
    + `SynchronousQueue`：无储存空间的无界队列，需要有消费线程和生产线程一一对应进行处理。
    
+ 饱和策略/拒绝策略

    如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了，
    `ThreadPoolExecutor`类中定义了4种拒绝策略。
    + AbortPolicy：默认策略。直接抛弃，抛出`RejectedExecutionException`来拒绝新任务的处理。
    + CallerRunsPolicy：用调用者的线程执行任务。直接在调用`execute()`方法的线程中运行被拒绝任务，直接运行`run()`方法。
        如果线程池执行程序已关闭，那就舍弃该任务。适用于任一任务都不能丢弃的情况。
    + DiscardPolicy：不处理新任务，直接丢弃掉。
    + DiscardOldestPolicy：此策略将丢弃最早的未处理的任务请求。
    
    也可以通过自定义`handler`实现`RejectedExecutionHandler`接口，根据需要编写拒绝策略。

+ 执行过程

    阅读`execute()`方法源码，来了解提交一个任务到线程池的过程：
    ```
    // 存放线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount)
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
    // 运行状态 runState
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
    
    private static int workerCountOf(int c) {
        return c & CAPACITY;
    }
    // 工作队列
    private final BlockingQueue<Runnable> workQueue;
    
    public void execute(Runnable command) {
        // 任务不为null，为null时排除异常。
        if (command == null)
            throw new NullPointerException();
        // 获取线程池当前状态信息
        int c = ctl.get();
        // 首先判断当前线程池中执行的任务数量是否小于 corePoolSize
        if (workerCountOf(c) < corePoolSize) {
            // 如果小于的话，通过addWorker(command, true)新建一个线程，
            // 并将任务(command)添加到该线程中；然后，启动该线程从而执行任务。
            // 线程池是非运行状态、workerCount超过最大容量或者超过最大线程数、线程启动失败时返回false
            // 这里用的是corePoolSize数
            if (addWorker(command, true))
                return;
            // 只有上面false时，更新一下线程池当前状态信息
            c = ctl.get();
        }
        // 如果当前之行的任务数量大于等于 corePoolSize 的时候就会走到这里
        // 判断线程池状态，只有RUNNING状态且工作队列可加入时，任务才会加入队列
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            // 再次检查线程池状态。
            // 如果线程池状态不是 RUNNING 状态就需要从任务队列中移除任务，
            // 并尝试判断线程是否全部执行完毕。同时执行拒绝策略
            if (! isRunning(recheck) && remove(command))
                reject(command);
            // 如果当前线程池为空，那就创建一个新的线程执行    
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        // 通过addWorker(command, false)新建一个线程，并将任务(command)添加到该线程中；然后，启动该线程从而执行任务。
        // 如果addWorker(command, false)执行失败，则通过reject()执行相应的拒绝策略的内容。
        // 这里用的是maximumPoolSize数
        else if (!addWorker(command, false))
            reject(command);
    }    
    ```
    + 过程：开始提交任务
        + 判断核心线程池是否已满？ 
            + 未满。创建线程。（实际线程数量 < corePoolSize）
            + 已满。判断等待队列是否已满？   
                + 未满。加入队列。（corePoolSize < 实际线程数量 < maximumPoolSize）
                + 已满。判断线程池是否已满？     
                    + 未满。创建线程。（实际线程数量 < maximumPoolSize）
                    + 已满。执行拒绝策略。（实际线程数量 > maximumPoolSize）

+ 设置合理的线程池大小
    
    + CPU 密集型任务(N+1)： 这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1，
                    比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，
                    或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，
                    而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。
    + I/O 密集型任务(2N)： 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，
            而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。
            因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。
    + 需考虑上下文切换（时间片轮转）
    
+ 关闭线程池：`shutdown()`（等待全部执行完成）和`shutdownNow()`（立即停止）
___ 
### ScheduledThreadPoolExecutor
`ScheduledThreadPoolExecutor`主要用来在给定的延迟后运行任务，或者定期执行任务。但实际基本不会用到。这里做了解学习。

`ScheduledThreadPoolExecutor`采用的任务队列是`DelayedWorkQueue`，它是基于堆结构（类似`DelayQueue`和`PriorityQueue`的数据结构）。

它会对队列中的任务进行排序，执行所需时间短的放在前面先被执行(`ScheduledFutureTask`的`time`变量小的先执行)，
如果执行所需时间相同则先提交的任务将被先执行(`ScheduledFutureTask`的`squenceNumber`变量小的先执行)。

在实际情况中推荐使用`Quartz`进行任务调度。
___
### Executors
Executors类中包含了很多方法。（待完善）
#### Executors提供的线程池创建方法
Executors提供了3种既定的线程池创建方法，但不建议这么使用，而是通过`ThreadPoolExecutor`构造函数的方式：
+ `Executors.newCachedThreadPool()`：无限线程池。
+ `Executors.newFixedThreadPool(nThreads)`：创建固定大小的线程池。
+ `Executors.newSingleThreadExecutor()`：创建单个线程的线程池。

不建议使用的原因：
+ `FixedThreadPool`和`SingleThreadExecutor`： 允许请求的队列长度为`Integer.MAX_VALUE`,可能堆积大量的请求，从而导致OOM。
+ `CachedThreadPool`和`ScheduledThreadPool`： 允许创建的线程数量为`Integer.MAX_VALUE`，可能会创建大量线程，从而导致OOM。

1. FixedThreadPool 

    可重用固定线程数的线程池。采用指定`nThreads`作为线程数，`corePoolSize`和`maximumPoolSize`都为`nThreads`。
    
    由于使用无界队列时`maximumPoolSize`将是一个无效参数，因为不可能存在任务队列满的情况。不会出现需要拒绝策略的情况。
    在任务很多的时候，容易出现OOM（内存溢出）。
    
2. SingleThreadExecutor 

    只有一个线程的线程池。`corePoolSize`和`maximumPoolSize`都为1。和FixedThreadPool有一样的问题。
    
3. CachedThreadPool 

    会根据需要创建新线程的线程池。`corePoolSize`为0，`maximumPoolSize`为`Integer.MAX_VALUE`。
    如果提交速度高于处理速度就会不断创建新的线程。极端情况下，这样会导致耗尽CPU和内存资源，从而导致OOM。
___ 