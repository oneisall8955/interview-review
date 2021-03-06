# 集合与JUC笔记

## 目录

- [基础集合扫盲](#基础集合扫盲)
    - [Collection](#Collection)
        - [List](#List)
        - [Map](#Map)
        - [Set](#Set)
- [JUC(java.util.concurrent)扫盲](#JUC扫盲)
    - [JUC关键点——volatile/CAS/AQS](#JUC关键点)
    - [AbstractQueuedSynchronizer](#AbstractQueuedSynchronizer)
        - [ReentrantLock](#ReentrantLock)
        - [Condition](#Condition)
        - [AQS组件](#AQS组件)
    - [Atomic原子类](#Atomic原子类)
        - [基本类型原子类](#基本类型原子类)
        - [数组类型原子类](#数组类型原子类)
        - [引用类型原子类](#引用类型原子类)
        - [对象的属性修改类型](#对象的属性修改类型)
        - [解决高并发性能问题——LongAdder/DoubleAdder](#可用于解决高并发性能问题的类)
    - [并发容器](#并发容器)
        - [ConcurrentHashMap](#ConcurrentHashMap)
        - [CopyOnWriteArrayList](#CopyOnWriteArrayList)
        - [ConcurrentLinkedQueue](#ConcurrentLinkedQueue)
        - [BlockingQueue](#BlockingQueue)
        - [ConcurrentSkipListMap](#ConcurrentSkipListMap)
    
___
## 基础集合扫盲
### Collection

Collection是所有集合的基类（接口）。
Java 8支持lambda语法。新增了不少的方法。（待拓展）
___
#### List

List是一个有序可重复的集合，允许有null值。
+ List提供了一个特殊的迭代器`ListIterator`。
+ 提供了`add()`, `remove(Object o)`, `addAll()`, `removeAll()`, `clear()`的新增删除操作方法。
+ 提供了`contains()`, `containsAll()`的对比方法。
+ 提供了`get()`, `set()`的查询方法
        
**Iterator**迭代器为集合提供的迭代器，只能单向移动，用于遍历集合。`Iterator.remove()`是**唯一安全**的方式来在迭代过程中修改集合。
___
1. **ArrayList**

    有序列表。实现`List`接口。底层为数组（`Object[]`）。
    + 初始化默认长度为10，超过默认长度时动态扩容。每次扩容生成扩容后的数组，将原数组数据复制到新数组，将新数组赋给原数组。
    
        扩容方法：`int newCapacity = oldCapacity + (oldCapacity >> 1);`
                    
        在remove删除时，每删一个数据，该数据后的所有数据将向左移动。
            
    + **ListIterator**
            
        List专有迭代器`ListIterator`。使用listIterator()生成。`ListIterator`的特性：
        - 双向移动（向前/向后遍历）
        - 产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引
        - 可以使用`set()`方法替换它访问过的最后一个元素
        - 可以使用`add()`方法在`next()`方法返回的元素之前或`previous()`方法返回的元素之后插入一个元素
                
    + **Iterator** 与 **ListIterator**的区别
                    
        1. Iterator 为通用迭代器，ListIterator 为List专有迭代器。
        2. Iterator 为单向遍历，ListIterator 可双向遍历。
        3. ListIterator 可以定位当前索引位置，Iterator 不支持。
        4. ListIterator 支持对遍历对象的修改，Iterator 不支持。
                
    + **Java 8 新增（待拓展）**
    
        Collection接口新增。
        + `forEach(Consumer<? super E> action)` (涉及Lambda)
                    
        + `spliterator()`(涉及Java Streams和Lambda)
                    
        Spliterator 为并行遍历数据源中的元素的迭代器，基于索引的、二分的、懒加载器。
                
2. **LinkedList**
    
    双向链表。实现`List`接口, `Deque（双端队列）`接口。非线程安全，在多线程环境下操作需进行同步。
        
    特点：新增、删除效率高。遍历时，为从头逐个遍历，效率低。
            
    + 经典问题：ArrayList和LinkedList的区别
        
3. Vector
        
    类似ArrayList，线程安全。实现了List接口。大部分方法都进行了同步。已弃用。

4. **Collections.synchronizedList(List list)**
        
    Collections提供了将集合转为线程安全的集合。Collections内置`SynchronizedList`类实现 List 接口方法时，内部都加上了 synchronized 代码块。
    但 Iterator 没有实现同步，所以在使用时需要对 Iterator 进行同步。
___
#### Map

Key-Value键值对（一对一）数据结构。Key是唯一的。
+ 提供了`put()`, `remove()`, `putAll()`, `removeAll()`的新增删除方法。
+ 提供了`keySet()`, `values()`, `entrySet()`获取键、值集合的方法。
        
1. **HashMap**

    `AbstractMap`类，实现了`Map`接口，允许key和value为null。非线程安全。
    + 默认初始容量为 16，负载因子为 0.75。
    + 默认最大容量为 `1<<30（2^30）`。超过默认最大容量后的最大容量为`Integer.MAX_VALUE`。
    + 刁钻问题：为什么是 30 而不是 31 呢？
    + 采用了**数组 + 链表**的结构
                    
    + **扩容机制**
    
        当map中填满了 负载因子*容器容量 数量的时候，map将进行扩容，扩容后数组的大小为原来HashMap大小的两倍。
        比如容量为16，负载因子为0.75，则在数组装满12（16*0.75=12）个元素的时候进行扩容，扩容后数组大小为32。
        map扩容后，对所有元素重新hash计算，再形成新的map。
                    
        **HashMap在扩容时性能消耗大。** 
        
        在存储数据大小越来越大时，在每次扩容都会对所有数据重新进行哈希计算，性能消耗极大。
        所以建议，如果可以预知HashMap中元素个数，在初始化的时候时填充HashMap容量（结合`负载因子*容器容量`和`2^n`个）可以提升性能。
                    
    + 哈希碰撞
                
        哈希计算公式：`hash = key.hashcode & (length - 1)`。当哈希值相同时，遵循先来后到的原则，以链表的形式链接。
                    
        注：`&`运算规则为两个数以二进制的形式进行比较，两个位都是1，结果才是1。
                    
    + **线程不安全**
                
        因为HashMap的扩容机制，存在可能多个线程对一个HashMap进行操作。
                    
        当多个线程对同个HashMap在进行插入操作并且HashMap需要进行扩容时，存在出现循环引用的问题。
    
    + JDK 1.8 增加 一定情况将链表结构转为红黑树
        增加了`TREEIFY_THRESHOLD`和`UNTREEIFY_THRESHOLD`两个参数。
        
        ```
        static final int TREEIFY_THRESHOLD = 8;         // 链表长度大于等于时，链表进化成树
        static final int UNTREEIFY_THRESHOLD = 6;       // 链表长度小于等于时，树退化成链表
        ```
    
                
2. **TreeMap**
    
    有序的、基于红黑树实现的Key-Value集合。继承`AbstractMap`类，实现了`NavigableMap`接口（继承关系`NavigableMap->SortedMap->Map`），支持一系列的导航方法。
    + 基于**红黑树**实现。其键按照**自然顺序**进行排序，或传入自定义`Comparator`进行排序。
    + 提供了`containsKey()`、`get()`、`put()` 和 `remove()`操作，其时间复杂度是 `log(n)` 。
    + 非线程安全。
                
3. **Hashtable**
        
    继承了`Dictionary`类，实现`Map`接口。类似`HashMap`，线程安全。不允许`key`和`value`为null。
    + 作为Hashtable的键对象需要实现`hashCode()`和`equals()`方法。
    + 采用`size()`, `isEmpty()`, `keys()`, `elements()`, `contains()`, `containsKey()`, `get()`, 
        `put()`, `remove()`, `putAll()`, `clear()`, `clone()`, `toString()`, `equals()`, `hashCode()`等
        对集合元素进行操作的操作时进行同步，用`volatile`修饰`keySet`, `entrySet`, `values`三个存储集合。
    + 线程安全，所以操作效率较低。
    + 经典问题：HashMap、Hashtable和ConcurrentHashMap的区别

4. **Collections.synchronizedMap(Map map)**
            
   Collections类提供了将Map转为线程安全的Map集合。Collections内置了`SynchronizedMap`类在进行操作的时候添加了synchronized同步代码块。
   同样，Iterator 没有实现同步，所以在使用时需要对 Iterator 进行同步。
            
   __**建议使用ConcurrentHashMap类。**__
___
#### Set

无序不重复的集合。底层数据结构使用 Map 实现。将Map的Key作为Set集合的内容，保证了唯一性。
        
+ HashSet: 底层使用 HashMap。
+ TreeSet: 自然排序的Set，底层使用 TreeMap。加入的元素必须实现 `Comparable` 接口。
+ `Collections.synchronizedSet(Set set)`与前面一样。

___
+ [返回顶部](#目录)
___
## JUC扫盲

`java.util.concurrent`包 简称 JUC 包。JUC离不开 **CAS(Compare And Swap)**、**volatile**和**AQS(AbstractQueuedSynchronizer)**。
`CAS`提供了一种无须加锁即可进行同步的思想，
`volatile`提供了访问变量的可见性，在很多场景下需要使用到。
`AQS(AbstractQueuedSynchronizer)`提供了一种实现阻塞锁和一系列依赖FIFO等待队列的同步器的框架。
JUC中很多类都是基于AQS构建的，如`ReentrantLock`，AQS使用`ReentrantLock`协助了解。
需要了解`CAS`、`volatile`、`ReentrantLock`几个基础知识。
### JUC关键点
1. **CAS(Compare And Swap)**

    比较并交换，是CPU硬件级别提供的功能。在JUC中使用该技术实现乐观锁。
    
    + **乐观锁**
    
        即认为别人取数据时，不会修改操作对象的数据，所以不加锁。适合读多写少的场景。
        
        在更新的时候判断期间有没有对这个数据进行更新，有则不做更新，没有则更新。
        
        注：以下缺点为原始问题具有。JDK 1.5后提供了解决方案。
        + ABA问题：初次读取为A值，准备赋值时仍是A，但期间有其他线程对其操作将A改成其他后再改回A。误认为从来没有被修改过。
        + 循环时间开销大：自旋CAS（不断重试直到成功）如果长时间不成功，CPU开销很大。
        + 只能保证一个共享变量的原子操作。
        
    + **悲观锁**
        
        即认为别人在取数据时，都会修改操作对象的数据，所以每次操作都加锁。适合写多读少的场景。
        
        Java中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

2. **volatile**

    类型修饰符。作用是作为指令关键字，确保本条指令不会因编译器的优化而省略。

    原子性操作：简单的读取、赋值（非变量间赋值）
   
    1. 特点：
    
        + 可见性。保证不同线程对修饰变量进行操作的可见性，即一个线程修改了某个变量的值，新的值对其他线程是立即可见的。
        + 有序性。禁止进行指令重排序。
        + 原子性。只能保证对单次读/写的原子性。
    
        `synchronized`和`Lock`也能保证可见性。
        保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。
    
    2. 原理和实现机制（保证可见性和禁止指令重排序）
    
        加入`volatile`关键字时，会多出一个`lock`前缀指令。
        
        `lock`前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：
        + 确保在执行到内存屏障这句指令时，在它前面的操作已经全部完成。即不会把内存屏障前面的指令排到其后面，不会把其后面的指令排到其前面。
        + 强制对缓存的修改操作立即写入主存中。
        + 写操作会导致其他CPU对应的缓存行失效。
        
    3. 应用场景
        + 对变量的写操作不依赖于当前值。
        + 该变量没有包含在具有其他变量的不变式中。
        
        只有在状态真正独立于程序内其他内容时才能使用 `volatile`，包括变量的当前状态。

3. **AQS**：即`AbstractQueuedSynchronizer`，这个类在`java.util.concurrent.locks`包下面。
    因为涉及到的类和学习的方法比较多，所以独立到下面的点进行学习。
___
### AbstractQueuedSynchronizer
AQS 是一个用来构建锁和同步器的框架。使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，
比如`ReentrantLock`，`Semaphore`，其他的诸如`ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask(jdk1.7)`等等皆是基于`AQS`的。
+ 核心思想
        
    如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
    如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，
    这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。
        
    `CLH(Craig,Landin,and Hagersten)`队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。
    `AQS`是将每条请求共享资源的线程封装成一个`CLH`锁队列的一个结点（Node）来实现锁的分配

+ 资源共享方式
    + 独占（Exclusive）：只有一个线程能执行，如`ReentrantLock`（见下 4. **ReentrantLock**），分为公平锁和非公平锁。
    + 共享（Share）：多个线程可同时执行，如`Semaphore`/`CountDownLatch`。
    
+ AQS 结构
        
    ```
    // 头结点，可理解为 当前持有锁的线程
    private transient volatile Node head;
    // 阻塞的尾节点，每个新的节点都插入到最后，形成一个链表
    private transient volatile Node tail;
        
    // 当前锁的状态，0代表没有被占用，大于 0 代表有线程持有当前锁
    // 这个值可以大于 1，是因为锁可以重入，每次重入都加上 1
    private volatile int state;
        
    // 当前持有独占锁的线程，举个最重要的使用例子，因为锁可以重入
    // reentrantLock.lock()可以嵌套调用多次，所以每次用这个来判断当前线程是否已经拥有了锁
    // if (currentThread == getExclusiveOwnerThread()) {state++}
    private transient Thread exclusiveOwnerThread; //继承自AbstractOwnableSynchronizer
    ```

参考：[一行一行源码分析清楚 AbstractQueuedSynchronizer(一)](https://javadoop.com/post/AbstractQueuedSynchronizer "AbstractQueuedSynchronizer1")        
参考：[一行一行源码分析清楚 AbstractQueuedSynchronizer(二)](https://www.javadoop.com/post/AbstractQueuedSynchronizer-2/ "AbstractQueuedSynchronizer2")
        
#### ReentrantLock
`ReentrantLock`为可重入锁，实现了`Lock`接口。`ReentrantLock`和`synchronized`都是可重入锁。
+ 可重入锁：也叫做递归锁，当一个线程请求得到一个对象锁后再次请求此对象锁，可以再次得到该对象锁。
+ `ReentrantLock`包含了3个内部类：`Sync`、`NonfairSync`（非公平锁）、`FairSync`（公平锁）。
    + `Sync`：继承了`AbstractQueuedSynchronizer`。
    + `NonfairSync`：非公平锁，继承了`Sync`抽象类。非公平锁就是一种获取锁的抢占机制，是**随机获得锁**的，非公平锁可能使线程“饥饿”。
    + `FairSync`：公平锁，继承了`Sync`抽象类。表示线程获取锁的顺序是按照**线程加锁的顺序**来分配的，即先来先得的**FIFO**先进先出顺序。

+ Node结构

    `prev`和`next`用于实现阻塞队列的双向链表，这里的`nextWaiter`用于实现条件队列的单向链表（后面Condition使用）。
    ```
    volatile int waitStatus; // 可取值 0、CANCELLED(1)、SIGNAL(-1)、CONDITION(-2)、PROPAGATE(-3)
    volatile Node prev;
    volatile Node next;
    volatile Thread thread;
    Node nextWaiter;
    ```
    
    waitStatus的5个状态：
    + 初始值：0。没有状态。线程没有获取锁。
    + `CANCELLED`：1。取消状态。在超时或中断时，节点会被设置为`CANCELLED`，不参与后续竞争，从队列删除并等待GC回收。
    + `SIGNAL`：-1。后继节点的线程处于等待状态。等待获取同步，前节点是head，当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行。
    + `CONDITION`：-2。等待条件状态，处于等待条件队列中，Condition相关，通过signal方法将节点转移到同步（阻塞）队列。
    + `PROPAGATE`：-3。表示下一次共享式同步状态获取将会无条件地传播下去。
    
+ `lock()`
    
    `lock()`方法在能获得锁就返回 true，不能的话一直等待获得锁。
    `NonfairSync`的`lock()`方法比`FairSync`的`lock()`方法多了一处，
    对当前锁状态尝试CAS更新，成功了则当前线程独占该锁。非公平锁就体现在这里，其他线程可能在等待获取锁仍为唤醒，而新来的线程直接抢占了锁。
    如果抢不到锁就等待。
    
    先来看公平锁的lock()，公平锁的加锁lock()方法，按着顺序一步一步来。
    ```
    // FairSync的lock()
    final void lock() {
        acquire(1);     // 试一下
    }
    ```
    在lock()方法中调用的`acquire(1)`方法为`AbstractQueuedSynchronizer`（即AQS父类）类中的方法，
    ```
    // 判断线程能不能拿到锁，然后再看要不要线程排队
    public final void acquire(int arg) {
        // 尝试去拿锁 tryAcquire返回true获取锁成功，false失败。
        if (!tryAcquire(arg) &&     // 尝试失败了就去加队列
            // acquireQueued用于队列中的线程自旋地以独占且不可中断的方式获取同步状态，直到拿到锁之后再返回。
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();        // 当前线程中断，搁置，仍可以继续运行
    }   
    ```
    首先去尝试拿锁，获取锁成功返回true，失败返回false。
    ```
    // FairSync的tryAcquire()
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        // 获取锁状态
        int c = getState();
        // c == 0说明还没有线程持有锁
        if (c == 0) {
            // 判断是否在队列中有其他线程在当前线程前面
            // 如果前面没有等待线程的话就CAS尝试修改锁状态
            // 如果加锁失败，证明几乎同时有人提前一步拿到锁
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                // 标记当前线程已经拿到锁了
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 到这里判断当前线程是否为锁持有者，是的话证明锁可重入，需要累加state
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;   // 累加
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        // 前两个if都进入不了，那就是没拿到锁
        return false;
    }
    ```
    如果在`tryAcquire()`拿锁失败了，那就需要将当前线程加入到阻塞队列当中。
    执行`acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`方法。
    首先执行的是`addWaiter()`方法。
    ```
    // 此方法的作用是把线程包装成node，同时进入到队列中
    // 上面传入的参数mode是Node.EXCLUSIVE，代表独占模式
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        // 判断队列是否为空
        if (pred != null) {
            // 队列不为空，则将当前队尾设为当前节点的前驱节点
            node.prev = pred;
            // 尝试使用CAS将自己设置为队尾
            if (compareAndSetTail(pred, node)) {
                // 队尾设置成功，再将其连接起来，将线程加入队列中
                pred.next = node;
                return node;
            }
        }
        // 如果队列为空 或者 CAS设置队尾时竞争失败，于是采用自旋的方式入队，总会排进去的
        enq(node);
        return node;
    }
    ```
    如果在将线程添加到队列过程中，出现了队列为空或者设置队尾竞争失败的情况，则会采用自旋入队。
    调用的是`enq()`方法。
    ```
    // 采用自旋的方式入队
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            // 队列为空则初始化head节点
            if (t == null) { // Must initialize
                // 初始化head节点
                if (compareAndSetHead(new Node()))
                    // 成功初始化head节点后，将队尾节点指向头节点
                    tail = head;
                    // 初始化完成之后，线程还没加入，于是继续循环
            } else {
                // 此时将待加入线程的节点的前驱加点设置为原对尾节点，就是将新节点往后加，直到成功
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }    
    ```    
    在完成了addWaiter()操作之后，线程已经进入到了阻塞队列中，正常情况下返回的是false
    ```
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                // 取刚刚加入队列中的节点的前驱节点
                final Node p = node.predecessor();
                // 如果前驱为头节点，那就证明这个节点为队列中的第一个
                // 然后去尝试获取锁
                if (p == head && tryAcquire(arg)) {
                    // 获取锁成功之后，将这个节点设置为头节点，即锁的持有者
                    // 可以理解为出队，因为head是占有锁线程，不包含在等待锁的阻塞队列中
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 当前节点不是队头，或者获取锁失败
                // 判断是否挂起当前线程等待唤醒
                if (shouldParkAfterFailedAcquire(p, node) &&
                    // 如果前面返回true则需要挂起线程，再来下面执行挂起
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            // 在tryAcquire抛出异常时，failed为true，
            if (failed)
                cancelAcquire(node);
        }
    }
    ```
    在`acquireQueued()`对刚刚进入阻塞队列中的线程进行判断，挂起或唤醒线程获取锁。
    只有在当前线程不是队头或阻塞队列的队头竞争锁失败会调用`shouldParkAfterFailedAcquire()`方法。
    ```
    // 第一个参数pred是前驱节点，第二个参数node是代表当前线程的节点
    // 返回true，说明是正常情况，当前线程需要被挂起等待后面唤醒，去执行parkAndCheckInterrupt()方法
    // 返回false，说明当前不需要被唤醒，不直接挂起线程，可能存在当前节点的前驱节点就是头节点，需要的是尝试获取锁而不是挂起线程
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        // 前驱节点的 waitStatus == -1 ，说明前驱节点状态正常，
        // 当前线程需要挂起，直接可以返回true
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        // 前驱节点的 waitStatus大于0，说明前驱节点取消了排队
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            // 因为线程唤醒操作需要依赖前驱节点，所以需要帮当前节点找到靠谱的前驱节点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            // 进入到这里的时候说明，前驱节点的waitStatus可能是0，-2，-3
            // 每个加入队列的节点初始化时，waitStatus为0，前驱也是
            // 用CAS将前驱节点的waitStatus设置为Node.SIGNAL(也就是-1)
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        // 这里返回false之后，会再回到外面进行循环，再进来方法
        // 然后再进到第一个if中，返回true
        return false;
    }
    ```
    
    非公平锁的加锁lock()方法
    ```
    // NonfairSync的lock()
    final void lock() {
        // 对当前锁的状态尝试进行CAS更新
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }    
    ```     
    `NonfairSync`和`FairSync`分别重写了`tryAcquire()`方法，
    `tryAcquire()`方法都是获取了当前同步状态，并用CAS尝试更新这个状态来获取锁。
    而不同的是`FairSync`中多了`hasQueuedPredecessors()`方法，用于判断头节点是否为当前线程。
    ```
    // NonfairSync的tryAcquire()
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }    
    ```
+ `unlock()`：释放锁，还原状态。包括了判断是否完全释放锁、是否有重入，然后释放锁，唤醒后继节点
+ `lockInterruptibly()`：一个可以响应中断的获取锁的方法，可以用来解决死锁问题。

+ 公平锁和非公平锁
    + 非公平锁在调用`lock`后，首先就会调用`CAS`进行一次抢锁，如果这个时候恰巧锁没有被占用，
        那么直接就获取到锁返回了。
    + 非公平锁在 CAS 失败后，和公平锁一样都会进入到`tryAcquire`方法，在`tryAcquire`方法中，
        如果发现锁这个时候被释放了（state == 0），非公平锁会直接 CAS 抢锁，
        但是公平锁会判断等待队列是否有线程处于等待状态，如果有则不去抢锁，直接排到后面。
        
    公平锁和非公平锁就这两点区别，如果这两次 CAS 都不成功，那么后面非公平锁和公平锁是一样的，
    都要进入到阻塞队列等待唤醒。

    相对来说，非公平锁会有更好的性能，因为它的吞吐量比较大。当然，
    非公平锁让获取锁的时间变得更加不确定，可能会导致在阻塞队列中的线程长期处于饥饿状态。
    
+ 在并发环境下，加锁和解锁需要以下三个部件的协调：
    + 锁状态。标记锁是否被其他线程占有。
    + 线程的阻塞和解除阻塞。
    + 阻塞队列。把争抢锁的线程加入队列（实际是链表）等待，遵循FIFO。AQS 采用了 CLH 锁的变体来实现。

+ 中断：Java的中断属于状态标识。程序可以选择响应中断和不响应中断。
___
#### Condition
`Condition`是依赖于`ReentrantLock`的，不管是调用`await`进入等待还是`signal`唤醒，都必须获取到锁才能进行操作。
`Condition`属于条件队列，适用于“生产者-消费者”场景，在该场景中，用于存放“生产者”生产的对象的队列为阻塞队列（或叫同步队列）。
在同步队列中已满且暂未有消费时，其他线程就需要等待，而等待线程就是使用`Condition`存放。

`Condition`是依赖于`ReentrantLock`的`newCondition()`去创建`ConditionObject`内部类的，它只有2个Node节点属性，
分别是`firstWaiter`和`lastWaiter`。Node结构如上提到，主要用到`nextWaiter`。
在使用Condition时，经常会用到`await()`方法和`signal()`方法搭配使用，伺机将等待的线程加入“生产者-消费者”中参与。
下面来学习了解一下`await()`方法和`signal()`方法。

+ 从等待到唤醒：`await()`和`signal()`

    `await()`方法是可被中断的，不可被中断的是另一个方法`awaitUninterruptibly()`。
    下面是围绕`await()`展开，按照线程从等待到唤醒最后获取锁的顺序进行学习。
    ```
    public final void await() throws InterruptedException {
        // 判断线程是否是中断状态，是的话抛出异常
        if (Thread.interrupted())
            throw new InterruptedException();
        // 1. 将当前线程加入到条件队列中
        Node node = addConditionWaiter();
        // 2. 完全释放独占锁
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        // 3. 等待进入阻塞队列
        // 判断是否存在于阻塞队列中
        while (!isOnSyncQueue(node)) {  
            // 如果不存在于阻塞队列中，线程挂起，等待唤醒
            LockSupport.park(this);
            // 直到被唤醒就跳出循环
            // 4. 等待signal唤醒线程，转移到阻塞队列
            // 5. 唤醒后检查中断状态
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        // 6. 获取独占锁
        // acquireQueued返回false，则说明该节点的线程被中断，并且是在signal前中断，则会重新设置interruptMode
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;        
        // 如果node的nextWaiter不为空，说明在signal前被中断
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        // 7. 处理中断状态
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
    ```
    1. `addConditionWaiter()`：将当前线程的节点加入到条件队列。
        ```
        private Node addConditionWaiter() {
            Node t = lastWaiter;
            // If lastWaiter is cancelled, clean out.
            // 如果条件队列的最后一个节点存在且取消了，则将其清除出队
            // 在条件队列中只有CONDITION状态
            if (t != null && t.waitStatus != Node.CONDITION) {
                // 遍历整个条件队列，清除已取消等待的节点
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
            // 初始化Node，指定waitStatus为Node.CONDITION
            Node node = new Node(Thread.currentThread(), Node.CONDITION);
            // 队尾如果为空，就设置为第一个节点，否则就加入队尾
            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }
        ```
    2. `fullyRelease()`：完全释放独占锁。返回`release`之前的`state`值。如果执行失败，节点会被设置为取消状态。
        ```
        final int fullyRelease(Node node) {
            boolean failed = true;
            try {
                int savedState = getState();
                // 将当前状态作为release参数，完全释放锁，最后设置state为0。
                if (release(savedState)) {
                    failed = false;
                    return savedState;
                } else {
                // 如果返回false，说明state不为0，前后状态不一，则抛出异常
                    throw new IllegalMonitorStateException();
                }
            } finally {
                // 将节点设置为CANCELLED，等待清除
                if (failed)
                    node.waitStatus = Node.CANCELLED;
            }
        }
        ```
    3. 等待进入阻塞队列
    
        通过`isOnSyncQueue()`方法判断这个节点是否存在于同步（阻塞）队列，即判断是否已经从条件队列转移到阻塞队列了。
        在加入条件队列时，`waitStatus`会初始化为`CONDITION(-2)`。在转移到阻塞队列时`waitStatus`会变成0。
        ```
        final boolean isOnSyncQueue(Node node) {
            // 如果waitStatus还是CONDITION，说明还在条件队列中
            // 如果node.prev为空，则说明不在阻塞队列中，因为node.prev是在阻塞队列中才会用到的
            if (node.waitStatus == Node.CONDITION || node.prev == null)
                return false;
            // 如果node.next不为空，node都存在后继节点了，说明已经在阻塞队列中了。
            if (node.next != null) // If has successor, it must be on queue
                return true;
            return findNodeFromTail(node);
        }
        ```
        `findNodeFromTail()`方法是从阻塞队列的队尾开始从后往前遍历找，如果找到相等的，说明在阻塞队列，否则就是不在阻塞队列。
        `findNodeFromTail()`的执行结果是`isOnSyncQueue()`方法的返回值，找到了返回true。
        ```
        // 从阻塞队列的队尾往前遍历，如果找到，返回 true
        private boolean findNodeFromTail(Node node) {
            Node t = tail;
            for (;;) {
                if (t == node)
                    return true;
                if (t == null)
                    return false;
                t = t.prev;
            }
        }
        ```
    4. 等待signal唤醒线程，转移到阻塞队列。signal是由另一个线程来操作。就像生产者-消费者模式中，
    如果线程因为等待消费而挂起，那么当生产者生产了一个东西后，会调用 signal 唤醒正在等待的线程来消费。
        ```
        // 这里会唤醒等待最久的线程，将这个线程对应的 node 从条件队列转移到阻塞队列
        public final void signal() {
            // 判断是否持有当前独占锁。调用 signal 方法的线程必须持有当前的独占锁，否则抛出异常
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            // 获取条件队列的第一个节点，如果不为空，那就进行转移
            if (first != null)
                doSignal(first);
        }
        ```
        `doSignal()`方法是将条件队列中第一个需要转移的node找到，并尝试转移至阻塞队列。
        ```
        private void doSignal(Node first) {
            do {
                // 将准备出队first节点的下个节点设置为条件队列的第一个节点
                // 如果为空，则后面没有等待的节点了，将lastWaiter置空
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                // 将准备出队的first节点的nextWaiter置空，取消与条件队列的关系，准备转移至阻塞队列
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&       // 尝试将first节点转移至阻塞队列中
                     (first = firstWaiter) != null);
             // 上面的循环，如果转移不成功，则选择first下一个节点进行转移
        }        
        ```
        `transferForSignal()`方法将节点从条件队列转移到阻塞队列。转移成功返回true。只有该节点在signal之前取消才会返回false。
        ```
        final boolean transferForSignal(Node node) {
            /* 如果CAS更新waitStatus失败，则说明该节点已经被取消了，不需要转移，返回，转移下一个节点
             * If cannot change waitStatus, the node has been cancelled.
             */
            if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
                return false;
            // 自旋进入阻塞队列的队尾，返回node在阻塞队列中的前驱节点p
            Node p = enq(node);
            int ws = p.waitStatus;
            // ws > 0，说明在阻塞队列中的前驱节点已经取消等待锁，直接唤醒node对应线程
            // 节点入队后，需要把前驱节点的状态设为 Node.SIGNAL(-1)
            if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
                // 如果前驱节点取消或者 CAS 失败，唤醒线程
                LockSupport.unpark(node.thread);
            return true;
        }        
        ```
    5. 唤醒后检查中断状态。在signal完成之后，线程从条件队列转移到了阻塞队列，await方法恢复，准备获取锁。
        ```
        // 这里会有3种返回值
        // REINTERRUPT(1): 代表 await 返回的时候，需要重新设置中断状态
        // THROW_IE(-1): 代表 await 返回的时候，需要抛出 InterruptedException 异常
        // 0: 说明在 await 期间，没有发生中断
        private int checkInterruptWhileWaiting(Node node) {
                    // 判断线程的状态
            return Thread.interrupted() ?
                // 如果线程已经中断，调用transferAfterCancelledWait方法
                (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
                0;  // 线程没中断
        }    
        ```
        只有在线程已经中断时，才会调用`transferAfterCancelledWait()`方法。
        如果此线程在 signal 之前被取消，返回true。
        即使发生了中断，节点依然会转移到阻塞队列。
        ```
        final boolean transferAfterCancelledWait(Node node) {   
            // CAS更新waitStatus为0，如果更新成功，说明是说明是 signal 方法之前发生的中断
            // 如果 signal 先发生的话，signal 中会将 waitStatus 设置为 0
            if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
                // 将节点放入阻塞队列
                enq(node);
                return true;
            }
            /*
             * If we lost out to a signal(), then we can't proceed
             * until it finishes its enq().  Cancelling during an
             * incomplete transfer is both rare and transient, so just
             * spin.
             */
            // CAS更新失败，说明signal已经把waitStatus设为0
            // signal会将节点转移到阻塞队列，可能还没完成，循环等待至转移完成
            // 发生的情况较少
            while (!isOnSyncQueue(node))
                Thread.yield();
            return false;
        }    
        ```    
    6. 获取独占锁
        
        在线程唤醒之后，节点已经进入阻塞队列，准备获取锁。
        ```
        // node已经进入了阻塞队列，savedState是之前释放锁前的 state
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        ```
        `acquireQueued(node, savedState)`方法返回false时，代表当前线程获取了锁。
        如果返回true，说明被中断了，而且`interruptMode != THROW_IE`，说明在signal之前就发生中断了，
        这里将`interruptMode`设置为`REINTERRUPT`，用于待会重新中断。
        
        如果node在signal唤醒前被中断，说明该节点是取消状态，那就清理取消的节点。
        ```
        // node.nextWaiter不为空，说明在signal前就被中断了，signal会断开与条件队列的联系
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        ```
    7. 处理中断状态
    
        + 0：什么都不做，没有被中断过；
        + THROW_IE：await 方法抛出 InterruptedException 异常，因为它代表在 await() 期间发生了中断；
        + REINTERRUPT：重新中断当前线程，因为它代表 await() 期间没有被中断，而是 signal() 以后发生的中断
    
        ```
        private void reportInterruptAfterWait(int interruptMode)
            throws InterruptedException {
            // 在 await() 期间发生了中断，抛出异常
            if (interruptMode == THROW_IE)
                throw new InterruptedException();
                // 重新中断
            else if (interruptMode == REINTERRUPT)
                selfInterrupt();
        }
        ```
    + 总结
        
        `Condition`依赖于`ReentrantLock`，常用于“生产者-消费者”场景中，在“生产者-消费者”队列（下称阻塞队列）满时，
        通过`Condition`建立一个条件队列，使其他线程要进行“生产”或“消费”时在条件队列中等待空闲位置，
        以完成原线程的任务。当阻塞队列满了之后，有线程请求数据入队时无法入队，该线程使用await挂起加入
        条件队列，待其他线程使用signal唤醒。
___
#### AQS组件
AQS的共享模式使用（关于共享锁的内容，后面两个还需要再深入了解一下，看了一遍还是有点懵逼）
##### CountDownLatch
latch 的中文意思是门栓、栅栏。到了某个设置值的时候，才放开一起运行。

+ 构造方法：构造方法内部采用`Sync`内部类继承AQS作为数据结构，并将传入的数值作为状态值state，用于“开闸”。
当 state == 0 时“开闸”。

`CountDownLatch`主要使用`await()`和`countDown()`进行“关门”和“开闸”。它们需要不同的线程去调用。

+ `await()`：等待唤醒，阻塞。
    
    ```
    // CountDownLatch类中的方法
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    ```
    
    ```
    // AbstractQueuedSynchronizer类中的方法
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        // 响应中断，如果中断了就抛出异常
        if (Thread.interrupted())
            throw new InterruptedException();
        // 只要state不为0，就返回true
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
    
    // 只有当 state == 0 的时候，这个方法才会返回 1
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }        
    ```
    只要未“开闸”（未到线程一起运行的临界值），就会进到`doAcquireSharedInterruptibly()`方法获取共享锁，
    并且为可中断。
    ```
    private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
        // 把当前线程以SHARED形式入队，返回入队节点
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                // 寻找当前节点的前驱节点
                final Node p = node.predecessor();
                // 如果前驱节点是头节点，说明当前线程是在阻塞队列的第一个
                if (p == head) {
                    // 如果还没到临界值（state == 0），r都是-1
                    int r = tryAcquireShared(arg);
                    // 只有唤醒线程之后才会进入if
                    if (r >= 0) {
                        // 唤醒后的线程会将head节点设置为node（当前线程）
                        // 然后继续唤醒后继节点
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                // 第一次会把前驱节点设置为SIGNAL，重新循环
                if (shouldParkAfterFailedAcquire(p, node) &&
                    // 挂起线程，等待唤醒
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }    
    ```
    如果线程在挂起入队后等待唤醒的过程中，当前线程会设置前驱节点的`waitStatus`为`SIGNAL(-1)`。
    ```
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        // 正常情况下，前驱节点的waitStatus为0，会进入到设置前驱节点的`waitStatus`，返回false回到循环中
        // 第二次循环进来后，pred已经为SIGNAL， 返回true
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) { 
            // 清除取消的节点
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }    
    ```
    在线程挂起的时候，需要有别的线程执行`countDown()`来达到唤醒标准。
+ `countDown()`：达到标准就去唤醒等待线程。
    
    ```
    public void countDown() {
        sync.releaseShared(1);
    }
    
    public final boolean releaseShared(int arg) {
        // 只有 state == 0 时，才会进入if语句
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
    
    // 只有state减到0才会返回true，其他情况都是使 state = state - 1
    protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }    
    ```
    当达到开闸标准（state == 0）的时候，执行`doReleaseShared()`唤醒线程。
    ```
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            // 判断是否是空队列
            // 当队列为空或队列中没有需要唤醒的节点时不需要唤醒后继节点
            if (h != null && h != tail) {
                // 如果不是空队列，head的waitStatus为SIGNAL(-1)，在入队等待时设置的
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    // 尝试CAS更新头节点 waitStatue 为0
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    // 唤醒head的后继节点，即阻塞队列的第一个节点
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            // head节点被占领了就继续循环
            if (h == head)                   // loop if head changed
                break;
        }
    }    
    ```
___    
##### CyclicBarrier
`CyclicBarrier`为另外一个共享锁，意思为“可重复利用的栅栏”，它是`ReentrantLock`和`Condition`的组合使用。
等第一个栅栏一起通过后，再重新生成另一个栅栏。

+ 构造方法：传入指定值和通过栅栏前需要执行的操作
    
    ```
    public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }
    ```

（待完善）
___
##### Semaphore
类似一个资源池，每个线程需要调用`acquire()`方法获取资源，然后才能执行，
执行完后，需要`release`资源，让给其他的线程用。
如果池里的资源暂时没有可以使用的，线程将进入阻塞队列等待。
___
### Atomic原子类

原子类指一个操作是不可中断的。JUC包里的基础原子类分为4类：
+ 基本类型：使用原子的方式更新基本类型。
    + AtomicInteger：整型原子类
    + AtomicLong：长整型原子类
    + AtomicBoolean ：布尔型原子类
    
+ 数组类型：使用原子的方式更新数组里的某个元素。
    + AtomicIntegerArray：整型数组原子类
    + AtomicLongArray：长整型数组原子类
    + AtomicReferenceArray ：引用类型数组原子类
    
+ 引用类型
    + AtomicReference：原子更新引用类型
    + AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
    + AtomicMarkableReference：原子更新带有标记位的引用类型
    
+ 对象的属性修改类型
    + AtomicIntegerFieldUpdater：原子更新整型字段的更新器
    + AtomicLongFieldUpdater：原子更新长整型字段的更新器
    + AtomicReferenceFieldUpdater：原子更新引用类型里的字段
___
#### 基本类型原子类

包含了`AtomicInteger`整型原子类，`AtomicLong`长整型原子类和`AtomicBoolean`布尔型原子类。
三个类提供的方法几乎相同。以`AtomicInteger`为例：

1. `AtomicInteger`类常用方法
    + `public final int get()`：获取当前的值
    + `public final int getAndSet(int newValue)`：获取当前的值，并设置新的值
    + `public final int getAndIncrement()`：获取当前的值，并自增
    + `public final int getAndDecrement()`：获取当前的值，并自减
    + `public final int getAndAdd(int delta)`：获取当前的值，并加上预期的值
    + `boolean compareAndSet(int expect, int update)`：如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
    + `public final void lazySet(int newValue)`：最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。

2. **线程安全原理**
    ```
    // setup to use Unsafe.compareAndSwapInt for updates
    // 使用Unsafe.compareAndSwapInt(CAS)进行更新
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
       
    static {
        try {
            // 获取 字段value 相对Java对象的“起始地址”的偏移量，可以理解为找到内存地址
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    // 储存的关键值
    private volatile int value;
    ```
    `AtomicInteger`类利用 `CAS (Compare And Swap)` + `volatile`和`Unsafe`类的`native`方法来保证原子操作，避免了`synchronized`的高额开销，提升效率。
___
#### 数组类型原子类

包含了`AtomicIntegerArray`整形数组原子类，`AtomicLongArray`长整形数组原子类 和 `AtomicReferenceArray`引用类型数组原子类。
三个类提供的方法几乎相同。以`AtomicIntegerArray`为例：

`AtomicIntegerArray`类常用方法
+ `public final int get(int i)`：获取 index=i 位置元素的值
+ `public final int getAndSet(int i, int newValue)`：返回 index=i 位置的当前的值，并将其设置为新值：newValue
+ `public final int getAndIncrement(int i)`：获取 index=i 位置元素的值，并让该位置的元素自增
+ `public final int getAndDecrement(int i)`：获取 index=i 位置元素的值，并让该位置的元素自减
+ `public final int getAndAdd(int i, int delta)`：获取 index=i 位置元素的值，并加上预期的值
+ `public final boolean compareAndSet(int expect, int update)`：如果输入的数值等于预期值，则以原子方式将 index=i 位置的元素值设置为输入值（update）
+ `public final void lazySet(int i, int newValue)`：最终 将index=i 位置的元素设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
___
#### 引用类型原子类

基本类型原子类只能更新一个变量，如果需要原子更新多个变量，需要使用**引用类型原子类**。 

包含了`AtomicReference`原子更新引用类型，
`AtomicStampedReference`原子更新带有版本号的引用类型 和 
`AtomicMarkableReference`原子更新带有标记位的引用类型的更新器

1. AtomicReference

    通过CAS方法操作传入AtomicReference中的对象，调用方法完成操作。
    实现原理与 AtomicInteger 类中的 compareAndSet 方法相同。
    
2. AtomicStampedReference
    
    带整型版本号，解决CAS-ABA的问题。通过版本号可以知道引用类型在中途过程中更改了几次。
    
    ```
    public boolean compareAndSet(V expectedReference, V newReference,
                                        int expectedStamp,
                                        int newStamp) {
        Pair<V> current = pair;
            return
                expectedReference == current.reference &&   // 引用不变
                expectedStamp == current.stamp &&           // 版本号不变
                ((newReference == current.reference &&      // 新引用等于旧引用
                    newStamp == current.stamp) ||           // 新版本号等于旧版本号
                casPair(current, Pair.of(newReference, newStamp)));     // 构造新的Pair对象并CAS更新
    }
    
    private boolean casPair(Pair<V> cmp, Pair<V> val) {
        // 调用Unsafe的compareAndSwapObject()方法CAS更新pair的引用为新引用
        return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
    }
    
    ```
    
    + 使用版本号进行控制。每次CAS的同时检查版本号有没有变。
    + 不重复使用节点的引用。每次都新建一个Pair作为CAS比较对象，最后更新新的Pair对象引用作为新引用。
    + 直接操作元素而不是节点。外部传入元素值及版本号，而不是节点（Pair）的引用。
    
3. AtomicMarkableReference
    
    带布尔（boolean）类型版本号（或叫标记位），表示引用类型是否被更改过。
    其他与`AtomicStampedReference`类似。
___
#### 对象的属性修改类型

当需要原子更新某个类里的某个字段时，需要用到**对象的属性修改类型原子类**。

包含了`AtomicIntegerFieldUpdater`原子更新整型字段的更新器，
`AtomicLongFieldUpdater`原子更新长整形字段的更新器 和
`AtomicStampedReference `原子更新带有版本号的引用类型的更新器

原子地更新对象的属性的步骤：
1. 使用静态方法`newUpdater()`创建一个更新器。

    因为对象的属性修改类型原子类都是抽象类，每次都需要使用`newUpdater()`创建一个更新器，并设置待更新的类和属性。

2. 更新的对象使用`volatile`修饰符。保证在线程之间共享变量时保证立即可见，并且调用者能够直接操作对象字段。

+ 注意：
    + 字段必须是`volatile`的。访问权限修饰符`public/protected/default/private`调用者与操作对象字段的关系一致。建议使用`public`。
    + `AtomicIntegerFieldUpdater`和`AtomicLongFieldUpdater`只能修改`int/long`类型的字段，不能修改其包装类型`Integer/Long`，如果要修改包装类型就需要使用`AtomicReferenceFieldUpdater`。
    + 只能是可修改的实例变量，不能为`static`、`final`修饰。
___
#### 可用于解决高并发性能问题的类

在高并发场景下，使用`LongAdder`和`DoubleAdder`可以提升性能，但会**消耗更多的内存空间**。在并发较低的场景下，`LongAdder`和`AtomicLong`性能差不多。其父类为`Striped64`。
`LongAdder`和`DoubleAdder`原理一致，这里以`LongAdder`为例进行分析学习。

+ LongAdder

    内部有2个变量，一个是`base`变量，一个`Cell[]`数组。
    + `base`变量：非竞态条件下，直接累加到该变量上。
    + `Cell[]`数组：竞态条件下，累加个各个线程自己的槽`Cell[i]`中。
    
    核心方法：
    + `public void add(long x)`
        
        源码：
        
        ```
        public void add(long x) {
            Cell[] as; long b, v; int m; Cell a;
            // 初始化时cells为null，进入casBase方法，进行CAS操作，如果操作成功则不会进入if语句内
            // 如果在发生并发冲突时，casBase失败，进入if语句内
            if ((as = cells) != null || !casBase(b = base, b + x)) {
                // 没有竞争的标志 uncontended
                boolean uncontended = true;
                // 判断Cell[] as数组是否初始化过
                // 如果初始化过，根据当前线程的Hash值映射到Cell[]数组(即as)指定槽位。
                // 对指定槽位的数值进行CAS更新，更新成功则不会进入if语句。
                // 如果更新失败，则进入if语句，执行longAccumulate方法
                if (as == null || (m = as.length - 1) < 0 ||
                    (a = as[getProbe() & m]) == null ||         
                    !(uncontended = a.cas(v = a.value, v + x)))
                    longAccumulate(x, null, uncontended);
             }
        }
        ```
        > 只有从未出现过并发冲突的时候，`base`基数才会使用到。只要发生冲突，后续只针对`Cell[]`数组的单元对象。
        >
        > 只有在 `Cell[]`数组未初始化 和 `Cell[]`数组已初始化但发生在Cell单元内发生冲突时，会调用父类的`longAccumelate`去初始化`Cell[]`或扩容。
        
        特点：尽量减少热点冲突，不到最后万不得已，尽量将CAS操作延迟。
    
    + Striped64类中`final void longAccumulate(long x, LongBinaryOperator fn, boolean wasUncontended)`
    
        `longAccumulate`分了三种情况进行处理：
        
        + Cell[]数组已经初始化
            
            ```
            // 如果cells已经被初始化了
            if ((as = cells) != null && (n = as.length) > 0) {
                // 根据当前线程的hash值映射找到Cell单元
                if ((a = as[(n - 1) & h]) == null) {                        
                    // 没有加锁，证明Cell[]数组没有正在扩容
                    if (cellsBusy == 0) {       // Try to attach new Cell   
                        // 创建一个Cell对象，传入x
                        Cell r = new Cell(x);   // Optimistically create    
                            // 尝试加锁，成功后 cellsBusy 为 1
                            if (cellsBusy == 0 && casCellsBusy()) {         
                                // 创建标志位
                                boolean created = false;
                                try {               // Recheck under lock
                                    // 在持有锁情况下对之前的判断进行检查
                                    Cell[] rs; int m, j;
                                    if ((rs = cells) != null &&
                                        (m = rs.length) > 0 &&
                                        rs[j = (m - 1) & h] == null) {
                                        rs[j] = r;     // 检查一致的话才将创建的Cell对象传入
                                        created = true;     // 将创建标志位置为true
                                    }
                                } finally {
                                    cellsBusy = 0;      // 释放锁
                                }
                                if (created)    // 只有创建成功的时候才会跳出循环，否则继续循环
                                    break;
                                continue;           // Slot is now non-empty
                            }
                        }
                        collide = false;     // 没有冲突，不需要扩容
                    }
                    // wasUncontended表示前一次CAS操作是否成功，为传入参数。
                    // 如果失败了则将wasUncontended置为true，重新进入循环，计算线程hash值
                    else if (!wasUncontended)       // CAS already known to fail
                        wasUncontended = true;      // Continue after rehash
                    // 尝试CAS操作，更新成功就跳出循环
                    else if (a.cas(v = a.value, ((fn == null) ? v + x : fn.applyAsLong(v, x))))
                        break;
                    // 如果Cell数组大小（n）超过CPU核数，或者cells被修改，重置collide，不需要扩容，直接重试。
                    // 因为在最后会重算线程Hash值，去重新找新的槽位，直到槽位可用
                    else if (n >= NCPU || cells != as)
                        collide = false;            // At max size or stale
                    // 到这里证明，Cell数组（a）不为空，但有多个线程在a上竞争，需要扩容cells数组。
                    else if (!collide)
                        collide = true;
                    // 尝试加锁进行扩容
                    else if (cellsBusy == 0 && casCellsBusy()) {
                        try {
                            // 判断在加锁前是否被修改过
                            if (cells == as) {      // Expand table unless stale
                                Cell[] rs = new Cell[n << 1];       // 扩容大小为当前容量的2倍
                                for (int i = 0; i < n; ++i)
                                    rs[i] = as[i];
                                    cells = rs;
                            }
                        } finally {
                            cellsBusy = 0;      // 释放锁
                    }
                    collide = false;            
                    continue;                   // Retry with expanded table
                }
                h = advanceProbe(h);        // 重算线程Hash值
            }
            ```
        + Cell[]数组未初始化
            
            ```
            // Cell没有初始化且没加锁，对其尝试加锁，初始化cells数组
            else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
                // 初始化标志init
                boolean init = false;
                try {                           // Initialize table
                    if (cells == as) {
                        Cell[] rs = new Cell[2];        // 初始化大小为2（必须是2^n）
                        rs[h & 1] = new Cell(x);        // 根据当前线程hash值计算映射索引并将传入值赋予对应索引的槽位
                        cells = rs;                     // 再赋给cells数组
                        init = true;                    // 初始化完成
                    }
                } finally {
                    cellsBusy = 0;      // 释放锁
                }
                if (init)
                    break;      // 初始化完成之后跳出
                }
            ```
            
        + Cell[]数组正在初始化中
            ```
            // 在初始化过程中，有其他线程也进入了此方法，直接将x累加到base上
            else if (casBase(v = base, ((fn == null) ? v + x : fn.applyAsLong(v, x))))
                    break;                          // Fall back on using base
            ```
    
        变量参数说明：
        + cellsBusy：自旋锁标志位。0代表空闲，1代表已加锁。
        + NCPU：CPU核数。
    
    + `LongAdder.sum()`
    
        返回**当前调用时刻**累加的值。此返回值可能不是绝对准确的。在调用`sum()`时可能有其他线程正在计数。
        
        **只能获取某个时刻的近似值。** 这也就是LongAdder并不能完全替代LongAtomic的原因之一。
    
    参考：[Java多线程进阶（十七）—— J.U.C之atomic框架：LongAdder](https://segmentfault.com/a/1190000015865714 "JUC——LongAdder")
    
+ LongAdder的其他兄弟们

    `LongAccumulator`、`DoubleAdder`、`DoubleAccumulator`三个。
    
    + `LongAccumulator`：是`LongAdder`的增强版。`LongAdder`只能针对数值的进行加减运算，而`LongAccumulator`提供了自定义的函数操作
    + `DoubleAdder`和`DoubleAccumulator`：用于操作`double`原始类型。内部会通过一些方法，将原始的`double`类型，转换为`long`类型。其他和`LongAdder`完全一样
___
+ [返回顶部](#目录)
___
### 并发容器

JDK提供的并发容器大部分存在于`java.util.concurrent`包。以下先重点学习JUC下的5个集合。
+ ConcurrentHashMap：
+ CopyOnWriteArrayList
+ ConcurrentLinkedQueue
+ BlockingQueue
+ ConcurrentSkipListMap
___
#### ConcurrentHashMap
由于`Hashtable`在操作时使用`synchronized`对整个对象进行加锁（锁住了整个Hash表），导致`Hashtable`效率低下。而`HashMap`为非线程安全。
在JDK 1.5到1.7版本中，Java使用了**分段锁机制**实现`ConcurrentHashMap`。
在JDK 1.8版本中，采用了`CAS + synchronized`代替`Segment + ReentrantLock`实现。
此处对1.8版本进行源码学习。
+ JDK 1.7版本

    `ConcurrentHashMap`采用了`Segment[]`数组和`HashEntry`组成，结构为数组加链表。
    + `Segment`：`ConcurrentHashMap`内部类。继承`ReentrantLock`，是一种可重入锁，类似于HashMap的结构。
    + `HashEntry`：存储键值对数据。
    
    将数据分成多个`Segment`，为每个`Segment`段都配了锁。当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。

+ JDK 1.8版本

    实现了`ConcurrentMap`接口。不允许`key`和`value`为null。更像是`Hashtable`。采用了`CAS + synchronized`代替`Segment + ReentrantLock`
    
    + 初始化
    
        初始化大小默认为16，负载因子为0.75。与`HashMap`一致。实际初始化存放位置是在添加元素的时候。
        
        初始化过程中，采用 `CAS` 配合 `sizeCtl` 进行加锁，其他线程在遇到数组正在初始化会让出CPU，保证初始化过程。
        
        插入数据时，如果定位到的数组槽位为空，则采用 `CAS` 进行数组槽插入；
        如果位置已有数据，判断所在位置链接的是链表还是红黑树：
        + 如果是链表的话，判断key及其hash值是否一致，一致则替换，不一致则用链表形式链接。
        + 如果是红黑树的话，那就插入树结构。
        + 有key相同进行替代的话，返回值为旧值。
        
        如果当前节点数binCount超过8个，则进行数组扩容或把节点转为树。(在实际情况下比较难见到这种情况发生。)
        最后使用`addCount()`进行累积当前节点的数量，并判断是否需要扩容。
        
        插入数据使用的`put()`方法中，调用`putVal()`的代码：
        ```
        final V putVal(K key, V value, boolean onlyIfAbsent) {
            // key和value不能为null
            if (key == null || value == null) throw new NullPointerException();  
            // 获取key的hash值。spread()为ConcurrentHashMap中获取hash值的方法
            int hash = spread(key.hashCode());
            int binCount = 0;       // 用来计算在这个节点总共有多少个元素，用来控制扩容或者转移为树
            for (Node<K,V>[] tab = table;;) {
                Node<K,V> f; int n, i, fh;
                // 第一次put的时候需要初始化table
                if (tab == null || (n = tab.length) == 0)
                    tab = initTable();  // 见下一个方法
                // 根据hash值计算找位置，如果对应的位置为null，进入if语句内
                // 对当前位置进行CAS更新，插入成功则跳出循环，否则重试。
                else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                    if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                        break;                   // no lock when adding to empty bin
                }
                // 如果定位到的位置的hash值为MOVED，数组正在扩容的数据复制阶段。
                // 当前线程参与复制，通过允许多线程复制的功能，一次来减少数组的复制所带来的性能损失
                else if ((fh = f.hash) == MOVED)
                    tab = helpTransfer(tab, f); 
                else {
                    // 如果定位到的位置有元素，采用synchronized加锁
                    V oldVal = null;
                    synchronized (f) {
                        // 再判断定位位置的对象和之前取的对象是否一致
                        if (tabAt(tab, i) == f) {
                            // hash值（fh = f.hash）大于0，则表示为链表。当转换为树之后，hash值为-2
                            if (fh >= 0) {
                                binCount = 1;
                                for (Node<K,V> e = f;; ++binCount) {
                                    K ek;
                                    // 如果遍历找到key和key的hash值都一样的节点，将原来的值替换。
                                    if (e.hash == hash &&           // key的hash值相同
                                        ((ek = e.key) == key || (ek != null && key.equals(ek)))) {    // key相同
                                        oldVal = e.val;
                                        if (!onlyIfAbsent)      // 在put方法默认为false，当使用putIfAbsent的时候才会跳过
                                            e.val = value;
                                        break;      
                                    }
                                    Node<K,V> pred = e;
                                    // 如果key一样，hash不一样，则判断下个节点是否为空，是的话就加入变成当前节点的下个节点。
                                    if ((e = e.next) == null) {
                                        pred.next = new Node<K,V>(hash, key, value, null);
                                        break;
                                    }
                                }
                            }
                            // 表示已经转化成红黑树类型了
                            else if (f instanceof TreeBin) {
                                Node<K,V> p;
                                binCount = 2;
                                // 调用putTreeVal方法，将该元素添加到树中去
                                if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                                    oldVal = p.val;
                                    if (!onlyIfAbsent)
                                        p.val = value;
                                }
                            }
                        }
                    }
                    if (binCount != 0) {
                        // 当在同一个节点的数目达到TREEIFY_THRESHOLD（8个）的时候，则数组进行扩容或将给节点的数据转为tree
                        if (binCount >= TREEIFY_THRESHOLD)
                            treeifyBin(tab, i);     // 判断是扩容还是转树，数组和传入hash定位位置
                        if (oldVal != null)
                            return oldVal;
                        break;
                    }
                }
            }
            addCount(1L, binCount); // 计binCount数量
            return null;
        }
        ```
        在第一次插入数据的时候，tab是空的，所以需要进行初始化操作。调用`initTable()`方法：
        ```
        // 用来初始化table存储数据
        private final Node<K,V>[] initTable() {
            Node<K,V>[] tab; int sc;
            while ((tab = table) == null || tab.length == 0) {
                // 如果sizeCtl<0说明正在进行创建，直接让出CPU
                if ((sc = sizeCtl) < 0)
                    Thread.yield(); // lost initialization race; just spin
                // 获取锁，CAS更新sizeCtl
                else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        // 在检查一遍在获取锁前是否有其他进行初始化
                        if ((tab = table) == null || tab.length == 0) {
                            // 第一次初始化时，sc为0，所以默认为DEFAULT_CAPACITY（16），后续扩容则以sc为扩容大小
                            int n = (sc > 0) ? sc : DEFAULT_CAPACITY;       
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];     // 生成数组，大小为n
                            table = tab = nt;   
                            sc = n - (n >>> 2);     // sc = 0.75*n
                        }
                    } finally {
                        sizeCtl = sc;   // 设置下一次扩容的大小
                    }
                    break;  // 循环拜拜！
                }
            }
            return tab;
        }
        ```
        当链表长度超过默认值（8）的时候，就会调用`treeifyBin()`方法判断是需要进行扩容还是转为树结构。
        ```
        // 当同一节点超过8个时调用此方法判断是扩容还是转树
        private final void treeifyBin(Node<K,V>[] tab, int index) {
            Node<K,V> b; int n, sc;
            if (tab != null) {
                // 如果数组长度小于MIN_TREEIFY_CAPACITY（64）进行扩容
                if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                    tryPresize(n << 1);     // 进入下面的扩容方法
                // 确定索引位置不为空且为链表结构
                else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                    // 锁住当前链表对象
                    synchronized (b) {
                        // 判断加锁前后链表对象是否更改，前后一致就把链表转为树
                        if (tabAt(tab, index) == b) {
                            TreeNode<K,V> hd = null, tl = null;
                            for (Node<K,V> e = b; e != null; e = e.next) {
                                // 先将Node转为TreeNode链接起来
                                TreeNode<K,V> p = new TreeNode<K,V>(e.hash, e.key, e.val, null, null);
                                if ((p.prev = tl) == null)
                                    hd = p;
                                else
                                    tl.next = p;
                                tl = p;
                            }
                            // new TreeBin<K,V>(hd)在这里才构建红黑树。
                            setTabAt(tab, index, new TreeBin<K,V>(hd)); 
                        }
                    }
                }
            }
        }
        ```
        每次插入数据一系列操作完成后，会对其进行元素计数。也会在计数时判断是否需要扩容，需要的时候会触发扩容。
        ```
        // addCount()计数
        private final void addCount(long x, int check) {
            CounterCell[] as; long b, s;
            // 判断是否存在竞争，不存在的话CAS更新BASECOUNT数量
            // 如果在CAS更新失败，证明存在竞争，使用counterCells记录累加。
            if ((as = counterCells) != null ||
                !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
                CounterCell a; long v; int m;
                boolean uncontended = true;
                // 1. 判断counterCells计数表是否为空
                // 2. 如果counterCells不为空，判断counterCells随机位置的值是否为空
                // 3. counterCells随机位置的值不为空，则对其进行CAS更新，更新失败调用fullAddCount计数
                if (as == null || (m = as.length - 1) < 0 ||
                    (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                    !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                    fullAddCount(x, uncontended);
                    return;
                }
                // 链表长度小于等于1，不需要考虑扩容
                if (check <= 1)
                    return;
                // 统计ConcurrentHashMap元素个数
                s = sumCount();
            }
            if (check >= 0) {
                // 需要扩容，内部根据情况去调用transfer()方法
                Node<K,V>[] tab, nt; int n, sc;
                // 如果s >= sc，元素个数（包括已插入和有冲突的）超过了扩容阈值
                // 并且table不为空，长度不超过最大容量时，需要扩容
                while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                       (n = tab.length) < MAXIMUM_CAPACITY) {
                    int rs = resizeStamp(n);    // 特殊扩容戳
                    if (sc < 0) {
                        // 在tryPresize方法也有一样的判断
                        // 5个条件只要满足一个就不能帮助扩容，直接跳出循环
                        // 1. (sc >>> RESIZE_STAMP_SHIFT)!=rs 表示比较高RESIZE_STAMP_BITS位生成戳和rs是否相等，相同
                        // 2. sc == rs+1 表示扩容结束
                        // 3. sc == rs + MAX_RESIZERS 表示帮助线程数已经达到最大值了
                        // 4. (nt = nextTable) == null 表示扩容已经结束
                        // 5. transferIndex<=0 表示所有的transfer任务都被领取完了，没有剩余的hash桶来给自己自己好这个线程来做transfer
                        if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                            sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                            transferIndex <= 0)
                            break;
                        if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                            transfer(tab, nt);
                    }
                    else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                                 (rs << RESIZE_STAMP_SHIFT) + 2))
                        transfer(tab, null);
                    s = sumCount();
                }
            }
        }        
        ```
    + 扩容机制：
         
        扩容包括了**容量变大**和**数据迁移到新集合内**。
        这里的扩容指初始化后对数组容量进行扩大，在多线程环境下还有其他线程可以协助扩容复制。<br>
        当数组需要进行扩容时，会调用`tryPresize()`方法。
        - 如果数组还没扩容迁移，判断是否已经在扩容：
            + 如果还没有进行扩容，在调用`transfer()`时会初始化一个2倍原数组长度的Node数组，赋给nextTable用于告知其他线程正在进行扩容转移。
            + 如果已经在进行扩容了，当前线程就会尝试协助扩容复制转移，需要协助的时候，调用`transfer()`时把nextTable传入参与扩容转移。
        
        `transfer()`根据目标数组的长度判断，至少让每个线程负责长度为16的数组段，如果长度只有16，那就只让一个线程负责。
        
        扩容后数组数据迁移，是针对原数组上的数据进行迁移，对划分的负责区间范围大小逐个对数组进行遍历，
        比如原数组大小为32，只有一个线程参与，默认负责范围为16，就按照从(0,15)和(16,31)两个区间进行遍历，从后往前。
        
        然后从被分配负责的范围的标志位（ForwardingNode对象的位置）往前遍历，逐个对遍历对象按规则进行迁移。规则为当前节点hash值和原数组长度的 & 计算。
        根据结果将一条链表分为两条链表——低位和高位，低位的链表留在原位置，高位的链表放到当前位置index加上扩容长度对应的位置，
        对应放到nextTab，即转移完成后会合并到对象真正存储的位置table。
        
        转移完成后，将扩容转移后的数组更新到原对象，重置扩容转移涉及到的中间对象。扩容转移的过程完成了。
        
        ```
        // 扩容方法，大小总是 2^n
        // 传入的size是原来数组长度的2倍
        private final void tryPresize(int size) {
            // 如果size大于 MAXIMUM_CAPACITY（1<<30） 最大容量的一半，则用最大容量，
            // c 是一个 大于等于（size * 1.5 + 1）的2的幂次方数    
            int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
                tableSizeFor(size + (size >>> 1) + 1);
            int sc;
            while ((sc = sizeCtl) >= 0) {
                Node<K,V>[] tab = table; int n;
                // 判断是否初始化，tab为空时为putAll()使用
                if (tab == null || (n = tab.length) == 0) {
                    // 初始化一个大小为sizeCtrl和刚刚算出来的c中较大的一个大小的数组
                    n = (sc > c) ? sc : c;
                    // CAS更新，获取“锁”(此锁非锁)，将sizeCtl设为-1
                    if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                        try {
                            // 判断加锁前后的对象是否一致，一致的话就初始化一个大小为n的数组，sizeCtl为0.75*n
                            if (table == tab) {
                                @SuppressWarnings("unchecked")
                                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                                table = nt;
                                sc = n - (n >>> 2); 
                            }
                        } finally {
                            sizeCtl = sc;
                        }
                    }
                }
                // 待确认
                else if (c <= sc || n >= MAXIMUM_CAPACITY)
                    break;
                // 说明还没迁移节点
                else if (tab == table) {
                    // 生成一个唯一的扩容戳
                    int rs = resizeStamp(n);
                    // sc<0，也就是sizeCtl<0，说明已经有别的线程正在扩容了     
                    if (sc < 0) {
                        Node<K,V>[] nt;
                        // 在addCount方法也有一样的判断
                        // 5个条件只要满足一个就不能帮助扩容，直接跳出循环
                        // 1. (sc >>> RESIZE_STAMP_SHIFT)!=rs 表示比较高RESIZE_STAMP_BITS位生成戳和rs是否相等，相同
                        // 2. sc == rs+1 表示扩容结束
                        // 3. sc == rs + MAX_RESIZERS 表示帮助线程数已经达到最大值了
                        // 4. (nt = nextTable) == null 表示扩容已经结束
                        // 5. transferIndex<=0 表示所有的transfer任务都被领取完了，没有剩余的hash桶来给自己自己好这个线程来做transfer
                        if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                            sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                            transferIndex <= 0)
                            break;
                        // 当前线程尝试帮助此次扩容
                        if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                            transfer(tab, nt);
                    }
                    // 如果当前没有在扩容，那么rs肯定是一个正数，
                    // 通过rs<<RESIZE_STAMP_SHIFT 将sc设置为一个负数，+2 表示有一个线程在执行扩容
                    else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
                        transfer(tab, null);
                }
            }
        }    
        ```
        符合扩容转移的要求就可以进行扩容或协助扩容。
        ```
        // 扩容操作的核心在于数据的转移。转移复制到新集合table
        private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
            int n = tab.length, stride;
            // stride 表示步长，一个线程至少处理数组长度为16
            // 数组长度只有16的话，就只会有1个线程去扩容复制移动
            if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
                stride = MIN_TRANSFER_STRIDE; // subdivide range
            // 复制目标为nextTab，为空时进行初始化，长度为原来数组长度的2倍
            if (nextTab == null) {            // initiating
                try {
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                    nextTab = nt;
                } catch (Throwable ex) {      // try to cope with OOME
                    sizeCtl = Integer.MAX_VALUE;
                    return;
                }
                // 将初始化后的目标数组赋予当前对象的转移目标数组
                nextTable = nextTab;
                // 把原数组长度作为后面转移时的分割索引点，用于控制迁移位置
                transferIndex = n;
            }
            // nextn
            int nextn = nextTab.length;
            // ForwardingNode是正在被迁移的Node，它的key，value，next都为null
            // hash为MOVED（-1），其中有个nextTable属性指向新tab[]（即上面初始化的nextTab）
            ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
            boolean advance = true;
            boolean finishing = false; // to ensure sweep before committing nextTab
            for (int i = 0, bound = 0;;) {
                Node<K,V> f; int fh;
                while (advance) {
                    int nextIndex, nextBound;
                    // 第一个条件一直为true，finishing为true则迁移任务结束
                    if (--i >= bound || finishing)
                        advance = false;
                    // 上一次迁移的边界赋值给nextIndex，一旦transferIndex <= 0
                    // 说明原数组的所有位置的迁移都有相应的线程去处理了，该线程可以不用迁移了
                    else if ((nextIndex = transferIndex) <= 0) {
                        i = -1;
                        advance = false;
                    }
                    // 将nextBound赋值给transferIndex，nextBound = nextIndex - stride（上一个边界减去步长）
                    else if (U.compareAndSwapInt(this, TRANSFERINDEX, nextIndex,
                              nextBound = (nextIndex > stride ? nextIndex - stride : 0))) {
                        bound = nextBound;
                        // 上一个边界 - 1 变成下次开始迁移的位置
                        i = nextIndex - 1;
                        advance = false;
                    }
                }
                // i<0就是所有迁移任务完成了
                if (i < 0 || i >= n || i + n >= nextn) {
                    int sc;
                    // 所有迁移完成，把nextTable设为空，把转移复制完成的数组赋给table，sizeCtl为0.75*新tab.length
                    if (finishing) {
                        nextTable = null;
                        table = nextTab;
                        sizeCtl = (n << 1) - (n >>> 1);
                        return;
                    }
                    // 当前线程迁移任务完成 sizeCtl - 1
                    if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                        // 不相等说明还有其他线程没完成迁移，该线程结束任务
                        if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                            return;
                        // 如果相等，则说明说有线程都完成任务了，设置finish为true    
                        finishing = advance = true;
                        // 再次循环检查一下整张表
                        i = n; // recheck before commit
                    }
                }
                // 如果旧tab[i]为null，则放入之前创建的ForwardingNode对象
                else if ((f = tabAt(tab, i)) == null)
                    advance = casTabAt(tab, i, null, fwd);
                // 上面定位的数据不为空时，判断是不是该节点已经有线程在迁移，是的话去下个节点尝试迁移
                else if ((fh = f.hash) == MOVED)
                    advance = true; // already processed
                else {
                    // 迁移开始，对当前节点对象加锁。把一条链表拆成两条。
                    // 
                    synchronized (f) {
                        if (tabAt(tab, i) == f) {
                            // ln表示低位， hn表示高位
                            Node<K,V> ln, hn;
                            if (fh >= 0) {
                                // 迁移头的位置（就是这段区间的尾节点）是个临界点，迁移头的hash值与数组大小做 & 运算
                                int runBit = fh & n;
                                // 把当前位置节点作为开始，遍历其链表
                                Node<K,V> lastRun = f;
                                for (Node<K,V> p = f.next; p != null; p = p.next) {
                                    // 把当前Node的hash值和数组大小做 & 运算
                                    int b = p.hash & n;
                                    if (b != runBit) {
                                        runBit = b;
                                        lastRun = p;
                                    }
                                }
                                // 记录runBit以及lastRun，记录最后一个与前面不一样的节点
                                if (runBit == 0) {
                                    ln = lastRun;
                                    hn = null;
                                }
                                else {
                                    hn = lastRun;
                                    ln = null;
                                }
                                // 生成 ln链 以及 hn链
                                for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                    int ph = p.hash; K pk = p.key; V pv = p.val;
                                    if ((ph & n) == 0)
                                        ln = new Node<K,V>(ph, pk, pv, ln);
                                    else
                                        hn = new Node<K,V>(ph, pk, pv, hn);
                                }
                                // 通过CAS操作，让ln链不动，hn链放到 i + n 的位置，并且设置当前节点为fwd，表示已经被当前线程迁移完了
                                // 同样一个hash值在扩容前和扩容后得到不同的位置
                                // 因为呈2倍大小扩容，同样一个hash值扩容前得到的位置和扩容后得到的位置刚好是相差n个（即增加扩容的长度）
                                setTabAt(nextTab, i, ln);
                                setTabAt(nextTab, i + n, hn);
                                setTabAt(tab, i, fwd);
                                advance = true;
                            }
                            else if (f instanceof TreeBin) {
                                TreeBin<K,V> t = (TreeBin<K,V>)f;
                                TreeNode<K,V> lo = null, loTail = null;
                                TreeNode<K,V> hi = null, hiTail = null;
                                int lc = 0, hc = 0;
                                for (Node<K,V> e = t.first; e != null; e = e.next) {
                                    int h = e.hash;
                                    TreeNode<K,V> p = new TreeNode<K,V>
                                        (h, e.key, e.val, null, null);
                                    if ((h & n) == 0) {
                                        if ((p.prev = loTail) == null)
                                            lo = p;
                                        else
                                            loTail.next = p;
                                        loTail = p;
                                        ++lc;
                                    }
                                    else {
                                        if ((p.prev = hiTail) == null)
                                            hi = p;
                                        else
                                            hiTail.next = p;
                                        hiTail = p;
                                        ++hc;
                                    }
                                }
                                // 判断是否需要退化成链表
                                ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                    (hc != 0) ? new TreeBin<K,V>(lo) : t;
                                hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                    (lc != 0) ? new TreeBin<K,V>(hi) : t;
                                setTabAt(nextTab, i, ln);
                                setTabAt(nextTab, i + n, hn);
                                setTabAt(tab, i, fwd);
                                advance = true;
                            }
                        }
                    }
                }
            }
        }        
        ```
        看完了没有其他线程协助的扩容转移，下面来看看有其他线程参与协助扩容的情况。协助扩容转移可以加快整个扩容速度。
        ```
        // 帮助转移数据
        final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
            Node<K,V>[] nextTab; int sc;
            // 判断此时是否仍然在执行扩容，nextTab=null的时候说明扩容已经结束了
            if (tab != null && (f instanceof ForwardingNode) &&
                (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
                int rs = resizeStamp(tab.length);
                // 说明扩容还未完成的情况下不断循环来尝试将当前线程加入到扩容操作中
                while (nextTab == nextTable && table == tab &&
                       (sc = sizeCtl) < 0) {
                        // 5个条件只要满足一个就不能帮助扩容，直接跳出循环
                        // 1. (sc >>> RESIZE_STAMP_SHIFT)!=rs 表示比较高RESIZE_STAMP_BITS位生成戳和rs是否相等，相同
                        // 2. sc == rs+1 表示扩容结束
                        // 3. sc == rs + MAX_RESIZERS 表示帮助线程数已经达到最大值了
                        // 4. (nt = nextTable) == null 表示扩容已经结束
                       // 5. transferIndex<=0 表示所有的transfer任务都被领取完了，没有剩余的hash桶来给自己自己好这个线程来做transfer
                                            
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                        transfer(tab, nextTab);
                        break;
                    }
                }
                return nextTab;
            }
            return table;
        }
        ```
        变量说明：
        + sizeCtl：用于多线程同步的互斥变量。
            + 当sizeCtl < 0时，表示已经有线程正在初始化哈希表或哈希表正在扩容，此时，不能再进行操作。-N代表有N-1个线程在进行扩容操作。
            + sizeCtl == -1时，代表即将初始化。
            + sizeCtl > 0时，代表下一次进行扩容的大小。
        + MOVED：值为-1。代表数组扩容数据正在复制阶段。    
        + baseCount：在没有冲突时使用的计数器。作为竞争备用，由CAS更新。存在竞争时不使用它。
        + counterCells：用于在竞争的时候计算元素个数。值为空时，不存在竞争。
    
    + 获取集合大小：`size()`
    
        通过调用`sumCount()`方法得到。
        元素个数保存baseCount中，部分元素的变化个数保存在CounterCell数组中，
        通过累加baseCount和CounterCell数组中的数量，即可得到元素的总个数
    
    + 经典问题：ConcurrentHashMap和Hashtable、HashMap的区别
___
#### CopyOnWriteArrayList
在很多应用场景下，读操作可能远大于写操作。读操作并不会修改原来的数据，每次读取都可以不加锁。
`CopyOnWriteArrayList`在读写锁思想上更进一步。只有写写互斥。
在不同进程访问同一资源的时候，只有在写操作，才会去复制一份新的数据，否则都是访问同一个资源。

`CopyOnWriteArrayList`实现了`List`接口。适合**读多写少**的场景。

+ 初始化：默认初始化一个长度为0的数组array。
该数组只能有`getArray()`和`setArray()`调用，`default`修饰，只有同包的类才能调用。

+ 写数据
    
    在写数据的时候，采用`ReentrantLock`加锁的方式进行，
    + `add(E e)`
    
        通过复制一个比原长度多1的数组生成一个新的数组，再把新增的数据写入，
        再将复制得到的数组赋给原来的数组。
    + `add(int index, E element)`
    
        在指定index位置的数组插入，其他往后移动。
    
    + `remove()`：同理。先加锁，再找元素，再操作复制数组，完成之后就赋给原数组。

+ 遍历

    `CopyOnWriteArrayList`中使用`COWIterator`迭代器。实现了`ListIterator`接口，仅支持遍历，不支持新增、删除、修改操作。
___
#### ConcurrentLinkedQueue
采用链表结构的、非阻塞的、线程安全的无界队列。继承`AbstractQueue`类，实现`Queue`接口。
适合在对性能要求相对较高，同时对队列的读写存在多个线程同时进行的场景，
即如果对队列加锁的成本较高则适合使用无锁的 ConcurrentLinkedQueue 来替代
+ 初始化

    默认初始化一个链表节点`Node`，并作为队列头`head`和队列尾`tail`。`Node`为内部类，节点的所有操作都是采用CAS更新。
+ 入队

    因为`ConcurrentLinkedQueue`是一个无界队列，所以在插入的时候一直都是返回`true`。
    入队主要做了2件事：定位尾结点，采用CAS将入队节点设置为尾结点的next节点，失败则重试。
    + 因为存在多线程插队的可能性，tail节点并不一定为尾结点。而判断是否为尾结点是当前节点的`next`节点为`null`。
        在插入的时候采用CAS更新，直到入队成功。
    
    + tail的更新时机是通过p和t是否相等来判断的，即当tail节点和尾节点的距离大于等于1时，更新tail。
    
    ```
    // add()方法调用offer()方法
    public boolean offer(E e) {
        checkNotNull(e);
        final Node<E> newNode = new Node<E>(e);
        
        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            if (q == null) {
                // 说明p是尾结点，CAS更新p的next节点，尝试入队。
                // p is last node
                if (p.casNext(null, newNode)) {
                    // Successful CAS is the linearization point
                    // for e to become an element of this queue,
                    // and for newNode to become "live".
                    // 更新tail节点，p != t是当之前的尾节点和现在插入的节点之间有一个节点时
                    // 在并发高的情况下，p!=t很容易成立，很多时候会尝试CAS更新tail节点。
                    if (p != t) // hop two nodes at a time
                        casTail(t, newNode);  // Failure is OK.
                    return true;
                }
                // Lost CAS race to another thread; re-read next
            }
            // 在多线程环境下，如果同时存在入队和出队的操作，可能出现p == q的情况
            // 此时需要重新找新的head
            // 如果p!=q则去寻找尾结点
            else if (p == q)
                // We have fallen off list.  If tail is unchanged, it
                // will also be off-list, in which case we need to
                // jump to head, from which all live nodes are always
                // reachable.  Else the new tail is a better bet.
                p = (t != (t = tail)) ? t : head;
            else
                // Check for tail updates after two hops.
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }
    ```
+ 出队

    出队操作与入队操作原理一样。通过CAS更新头节点，但并不是head节点并不一定是头节点，而是中间间隔一个元素时更新。

    ```
    public E poll() {
        restartFromHead:
        for (;;) {
            // 从头开始，并获取头节点信息。
            for (Node<E> h = head, p = h, q;;) {
                E item = p.item;
                // 如果头节点信息不为空，则使用CAS更新为null
                if (item != null && p.casItem(item, null)) {
                    // Successful CAS is the linearization point
                    // for item to be removed from this queue.
                    // 与入队时更新尾结点原理一样，并不是每次都去更新头节点。
                    if (p != h) // hop two nodes at a time
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    return item;
                }
                // 获取p节点的下一个节点，如果p节点的下一节点为null，则表明队列已经空了
                else if ((q = p.next) == null) {
                    updateHead(h, p);
                    return null;
                }
                // 说明有其他线程更新了head头节点，需要获取新的头节点
                else if (p == q)
                    continue restartFromHead;
                else
                // 如果下一个元素不为空，则将头节点的下一个节点设置成头节点
                    p = q;
            }
        }
    }    
    ```
+ 获取队列元素个数近似值：`size()`
    
    因为`size()`方法没有加锁，只是遍历了整个队列，遍历过程可能存在出入队的操作，所以size是一个近似值而非精准值。
+ 无界队列：因为是无界队列，在使用时需要注意内存溢出的问题。
___
+ [返回顶部](#目录)
___
#### BlockingQueue
阻塞队列，被广泛使用在“生产者-消费者”问题中，其原因是`BlockingQueue`提供了可阻塞的插入和移除的方法。
当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止。

`BlockingQueue`是一个接口，继承自`Queue`，所以其实现类也可以作为`Queue`的实现来使用，而 Queue 又继承自 Collection 接口。
`BlockingQueue`不接受`null`值的插入。

实现类有`ArrayBlockingQueue`、`LinkedBlockingQueue`、`PriorityBlockingQueue`、`DelayQueue`、
`LinkedTransferQueue`、`SynchronousQueue`。主要学习前三个。

参考：[解读Java并发队列 BlockingQueue](https://www.javadoop.com/post/java-concurrent-queue "BlockingQueue")
___
##### ArrayBlockingQueue
`ArrayBlockingQueue`实现了`BlockingQueue`接口，满足FIFO特性，是阻塞的**有界队列**，底层采用**数组**来实现，是一个循环数组。一旦创建则容量无法更改。

+ 初始化

    创建`ArrayBlockingQueue`时必须指定数组大小。默认情况下不能保证线程访问队列的公平性。
    
    + 所谓公平性是指严格按照线程等待的绝对时间顺序，即最先等待的线程能够最先访问到`ArrayBlockingQueue`。
    + 而非公平性则是指访问`ArrayBlockingQueue`的顺序不是遵守严格的时间顺序，
    有可能存在，当`ArrayBlockingQueue`可以被访问时，长时间阻塞的线程依然无法访问到`ArrayBlockingQueue`。
    + 如果保证公平性，通常会降低吞吐量。
    
    ```
    // 其他构造函数的fair默认为false
    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);     // 可重入锁
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }    
    ```
+ 入队

    尝试获取锁，当得到锁之后，比较判断当前的元素个数与数组长度，当相等时，那么队列已经满了，无法插入。
    
    实际进行入队操作的方法为`enqueue()`，调用它的方法有`offer()`（为`add()`所调用），`put()`。：
    ```
    // 在调用enqueue方法时，外层已经加锁了，所以这里无需进行加锁同步操作。
    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        // putIndex为下一个放置元素的位置
        items[putIndex] = x;        
        // 判断加入元素之后长度是不是和数组长度一样
        // 是的话重置putIndex，从数组开头开始
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        // 唤醒等待“数组不空”线程
        notEmpty.signal();
    }    
    ```

+ 出队
    
    实际出队方法为`dequeue()`方法，调用它的方法有：`poll()`，`take()`：

    ```
    // 在调用dequeue方法时，外层已经加锁了，所以这里无需进行加锁同步操作。
    private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;        // 把取出来的元素原来的位置设为null
        if (++takeIndex == items.length)    // 已经取到了数组的末尾，那么就要从头开始取
            takeIndex = 0;
        count--;
        if (itrs != null)
            // 每当元素已出队时调用。因为是循环队列，需要保证迭代器定位的准确性。
            itrs.elementDequeued();
        // 唤醒等待“数组不满”线程
        notFull.signal();
        return x;
    }   
    ```    
    
___
##### LinkedBlockingQueue
继承了`AbstractQueue`类，实现了`BlockingQueue`接口，满足FIFO特性，是底层基于**单向链表**实现的阻塞队列。
可以当做无界队列使用，也可以当做有界队列来使用。
与`ArrayBlockingQueue`相比起来具有更高的吞吐量。

`ArrayBlockingQueue`队列在出队和入队时使用不同的锁，即出队入队都有各自的锁，提高了并发性。

+ 初始化

    为了防止`LinkedBlockingQueue`容量迅速增，损耗大量内存，
    通常在创建`LinkedBlockingQueue`对象时，会指定其大小，
    如果未指定，容量等于`Integer.MAX_VALUE`。

    ```
    // 默认构造函数，某种意义上的无界队列
    public LinkedBlockingQueue() { 
        this(Integer.MAX_VALUE); 
    }
    // 指定大小，则为有界队列
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }
    ```
+ 入队
    
    入队拥有自己的锁`ReentrantLock putLock`。
    入队操作包括了：put()、offer()两种方法。两种方法的区别：
    + put()方法如果队列满了会进入等待，直到插入成功或抛出异常，无返回值。
    + offer()方法在插入成功时返回true，如果队列满了返回false，不进行阻塞等待或者等待指定时间。
    
    put()方法：
    ```
    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        // Note: convention in all put/take/etc is to preset local var
        // holding count negative to indicate failure unless set.
        // 本地变量c，默认负数为失败
        int c = -1;
        Node<E> node = new Node<E>(e);
        // 插入和删除都有各自的锁
        final ReentrantLock putLock = this.putLock;
        // 获取当前元素计数器
        final AtomicInteger count = this.count;
        // 如果当前线程未被中断且锁空闲，则获取锁
        putLock.lockInterruptibly();
        try {
            /*
             * Note that count is used in wait guard even though it is
             * not protected by lock. This works because count can
             * only decrease at this point (all other puts are shut
             * out by lock), and we (or some other waiting put) are
             * signalled if it ever changes from capacity. Similarly
             * for all other uses of count in other wait guards.
             */
            // 如果队列满了，当前“插入”线程阻塞等待
            while (count.get() == capacity) {
                notFull.await();
            }
            // 等到队列不满了，入队，放到链表尾部
            enqueue(node);
            // CAS更新元素数，c还是旧值
            c = count.getAndIncrement();
            // 如果队列没满，那就唤醒正在等待“插入”的线程
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();       // 释放锁
        }
        // 代表队列在这个元素入队前是空的（不包括head空节点），唤醒等待“获取”的线程
        if (c == 0)
            signalNotEmpty();
    }
    ```
+ 出队    

    出队拥有自己的锁`ReentrantLock takeLock`。
    head节点始终是空的，出队时把head使用下一个节点，把节点值设为null，原本的head节点自循环引用自动删除。
    
    出队操作包含了：`take()`和`poll()`两种方法。它们的区别是：
    + take()方法如果队列为空则等待至队列不为空或抛出异常，并出队。
    + poll()方法出队成功则返回队列头部元素，否则返回null。
    
    take()方法：
    ```
    public E take() throws InterruptedException {
        E x;
        int c = -1;
        final AtomicInteger count = this.count; // 当前元素个数计数器
        final ReentrantLock takeLock = this.takeLock;   
        // 如果当前线程未被中断且锁空闲，则获取锁
        takeLock.lockInterruptibly();
        try {
            // 如果队列为空，当前“获取”线程阻塞进入等待
            while (count.get() == 0) {
                notEmpty.await();
            }
            // 等到队列不为空了，出队
            x = dequeue();
            c = count.getAndDecrement();    // 元素减一
            // 如果队列有至少一个元素了，唤醒等待“获取”线程
            if (c > 1)
                notEmpty.signal();
        } finally {
            takeLock.unlock();  // 释放锁
        }
        // 在这个 take 方法发生的时候，队列是满的，唤醒等待“插入”线程
        if (c == capacity)
            signalNotFull();
        return x;
    }    
    ```
+ 问题：ArrayBlockingQueue和LinkedBlockingQueue的区别
___
##### PriorityBlockingQueue
继承了`AbstractQueue`类，实现了`BlockingQueue`接口。是一个支持**优先级的无界阻塞**队列。
是一个阻塞的`PriorityQueue`（优先级队列）。
采用自然排序，不允许插入null元素和不可比较的对象。优先级体现在`Comparable`比较器上。


+ 初始化
    
    默认初始化一个大小为11的、采用自然排序的数组，基于数组的二叉堆来存放元素。
    + 二叉堆：一颗完全二叉树，它非常适合用数组进行存储，
        对于数组中的元素 a[i]，其左子节点为 a[2*i+1]，其右子节点为 a[2*i + 2]，
        其父节点为 a[(i-1)/2]，其堆序性质为，每个节点的值都小于其左右子节点的值。
        二叉堆中最小的值就是根节点，但是删除根节点是比较麻烦的，因为需要调整树。
        
    最小的元素一定是根元素，它是一棵满的树，除了最后一层，最后一层的节点从左到右紧密排列。

+ 自动扩容

    自动扩容发生插入元素时数组空间不够的情况，在扩容之前需要先释放之前插入时加的锁`ReentrantLock`，
    然后再进行CAS操作`allocationSpinLock`自旋锁加锁。在节点数较小的时候，数组扩容增长得快一些。
    
    ```
    private void tryGrow(Object[] array, int oldCap) {
        // 扩容是发生在插入元素的时候空间不够，在这之前已经加锁，所以需要把之前加的锁释放
        lock.unlock(); // must release and then re-acquire main lock
        Object[] newArray = null;
        // 获取锁，allocationSpinLock指自旋锁分配，0是空闲，1是加锁。
        if (allocationSpinLock == 0 &&
            UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset,
                                     0, 1)) {
            try {
                // 如果节点个数小于 64，那么增加的 oldCap + 2 的容量
                // 如果节点数大于等于 64，那么增加 oldCap 的一半
                // 所以节点数较小时，增长得快一些
                int newCap = oldCap + ((oldCap < 64) ?
                                       (oldCap + 2) : // grow faster if small
                                       (oldCap >> 1));
                if (newCap - MAX_ARRAY_SIZE > 0) {    // possible overflow 可能溢出
                    int minCap = oldCap + 1;
                    if (minCap < 0 || minCap > MAX_ARRAY_SIZE)
                        throw new OutOfMemoryError();
                    newCap = MAX_ARRAY_SIZE;
                }
                // queue!=array说明有其他线程为其分配了其他空间
                if (newCap > oldCap && queue == array)
                    newArray = new Object[newCap];  // 分配一个新的大数组
            } finally {
                allocationSpinLock = 0; // 释放锁
            }
        }
        // 如果有其他的线程也在做扩容的操作
        if (newArray == null) // back off if another thread is allocating
            Thread.yield();
        // 重新获取锁    
        lock.lock();
        // 将原来数组中的元素复制到新分配的大数组中
        if (newArray != null && queue == array) {
            queue = newArray;
            System.arraycopy(array, 0, newArray, 0, oldCap);
        }
    }    
    ```
+ 插入数据
    
    使用`add()/put()`方法进行插入数据，其中直接调用了`offer()`方法。
    
    ```
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        final ReentrantLock lock = this.lock;
        lock.lock();    // 加锁
        int n, cap;
        Object[] array;
        // 如果当前队列中的元素个数大于等于数组的大小，那么需要扩容了
        while ((n = size) >= (cap = (array = queue).length))
            tryGrow(array, cap);
        try {
            Comparator<? super E> cmp = comparator;
            // 节点添加到二叉堆中
            if (cmp == null)
                siftUpComparable(n, e, array);  // 默认比较器
            else
                siftUpUsingComparator(n, e, array, cmp);    // 自定义比较器
            // 更新 size
            size = n + 1;
            // 唤醒等待的读线程
            notEmpty.signal();
        } finally {
            lock.unlock();  // 释放锁
        }
        return true;
    }
    ```
    ```
    // 将数据 x 插入到数组 array 的位置 k 处，然后再调整树
    private static <T> void siftUpComparable(int k, T x, Object[] array) {
        Comparable<? super T> key = (Comparable<? super T>) x;
        while (k > 0) {
            // 二叉堆中a[k]节点的父节点位置
            int parent = (k - 1) >>> 1;
            Object e = array[parent];
            if (key.compareTo((T) e) >= 0)
                break;
            array[k] = e;   // 父节点要比子节点小
            k = parent;
        }
        array[k] = key;
    }    
    ```
+ 出队
    
    出队操作包含了：`take()`和`poll()`。它们的区别是：
    + `take()`方法在队列为空时阻塞等待，直到出队成功。
    + `poll()`方法直接返回出队操作，不会阻塞等待或等待指定时间。
    
    ```
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        E result;
        try {
            // 出队
            while ( (result = dequeue()) == null)
                notEmpty.await();
        } finally {
            lock.unlock();
        }
        return result;
    }
    ```
    `take()`和`poll()`调用了`dequeue()`方法：
    ```
    private E dequeue() {
        int n = size - 1;
        if (n < 0)
            return null;
        else {
            Object[] array = queue;
            E result = (E) array[0];    // 队头，用于返回
            E x = (E) array[n];         // 队尾元素先取出    
            array[n] = null;            // 队尾置空
            Comparator<? super E> cmp = comparator;
            if (cmp == null)
                siftDownComparable(0, x, array, n);     // 删除，调整二叉堆
            else
                siftDownUsingComparator(0, x, array, n, cmp);
            size = n;
            return result;
        }
    }
    ```
    因为二叉堆中，根节点为最小元素，所以出队时直接取根节点，对应的是数组的第一个元素，然后其他节点进行调整。
___
##### BlockingQueue的其他实现类
这里简单介绍一下实现BlockingQueue的其他几个类，`SynchronousQueue`、`LinkedTransferQueue`、`DelayQueue`。
###### SynchronousQueue
`SynchronousQueue`是个特殊的队列——同步队列。这里的同步是指读线程和写线程需要同步。
当一个线程往队列里写入一个元素时，不会立即返回，需要等待另一个线程将这个元素取走。
即一个读线程操作需要一个写线程操作匹配，反之亦然。
    
实现了`BlockingQueue`接口，`SynchronousQueue`是一个虚队列，没有存储元素的空间。对线程进行排队，不储存元素。
它在线程池的实现类`ThreadPoolExecutor`中得到了应用。
    
因为没有元素存储空间，`SynchronousQueue`避免使用`add()`/`offer()`/`peek()`（只读取不移除，即返回null）等立即返回的方法，也不能被迭代。
    
+ 初始化
    
    初始化时可以指定**公平模式**(队列)和**非公平模式**(栈)。默认为非公平模式。
    ```
    public SynchronousQueue(boolean fair) {
        // 公平模式：TransferQueue，遵循FIFO
        // 非公平模式：TransferStack 
        transferer = fair ? new TransferQueue() : new TransferStack();
    }
    ```
    其中`TransferQueue`和`TransferStack`为两个内部类，继承自内部类`Transferer`。
    
    ```
    abstract static class Transferer<E> {
        /**
         * Performs a put or take.
         *
         * @param e if non-null, the item to be handed to a consumer;
         *          e 不为 null，元素从生产者转移到消费者
         *          if null, requests that transfer return an item offered by producer.
         *          e 为 null，消费者等待生产者提供元素
         * @param timed if this operation should timeout
         * @param nanos the timeout, in nanoseconds
         * @return if non-null, the item provided or received; if null,
         *         the operation failed due to timeout or interrupt --
         *         the caller can distinguish which of these occurred
         *         by checking Thread.interrupted.
         */
        abstract E transfer(E e, boolean timed, long nanos);
    }    
    ```
+ 出入队    
    + 公平模式下的`transfer()`方法
        + 当调用这个方法时，如果队列是空的，或者队列中的节点和当前的线程操作类型一致
            （如当前操作是`put`操作，而队列中的元素也都是写线程）。
            这种情况下，将当前线程加入到等待队列即可。
        + 如果队列中有等待节点，而且与当前操作可以匹配
            （如队列中都是读操作线程，当前线程是写操作线程，反之亦然）。
            这种情况下，匹配等待队列的队头，出队，返回相应数据。 
    + 非公平模式下的`transfer()`方法
        + 当调用这个方法时，如果队列是空的，或者队列中的节点和当前的线程操作类型一致
            （如当前操作是`put`操作，而栈中的元素也都是写线程）。
            这种情况下，将当前线程加入到等待栈中，等待配对。然后返回相应的元素，
            或者如果被取消了的话，返回null。
        + 如果栈中有等待节点，而且与当前操作可以匹配
            （如栈里面都是读操作线程，当前线程是写操作线程，反之亦然）。
            将当前节点压入栈顶，和栈中的节点进行匹配，然后将这两个节点出栈。
            配对和出栈的动作其实也不是必须的，因为下面的一条会执行同样的事情。
        + 如果栈顶是进行匹配而入栈的节点，帮助其进行匹配并出栈，然后再继续操作。
    
+ 公平模式（FIFO）在竞争下支持更高的吞吐量，非公平模式（LIFO）在一般的应用中保证更高的线程局部性
___
###### LinkedTransferQueue
实现了`TransferQueue`接口。是基于**单链表**结构的**阻塞无界**队列。遵循FIFO。内部节点可以视为“生产者-消费者”，类似于`SynchronousQueue`。

`TransferQueue`接口继承了`BlockingQueue`接口并进行了扩充。增加了`tryTransfer()`和`transfer()`等方法

+ 初始化：默认构造函数是不做任何操作的。在插入数据的时候，才会生成链表结构存储，没有做长度限制，是无界队列。
+ 入队和出队

    入队和出队操作，与4个`NOW`、`ASYNC`、`SYNC`和`TIMED`常量和1个`xfer()`方法有关。
    + `NOW`：即时操作，可能失败，不会阻塞调用线程。
        + `poll()`：获取并移除队首元素，如果队列为空，直接返回null。
        + `tryTransfer()`：尝试将元素传递给消费者，如果没有等待的消费者，则立即返回false，也不会将元素入队。
    + `ASYNC`：异步操作，一定成功。    
        + `offer()`：插入指定元素至队尾，由于是无界队列，所以会立即返回，返回值为true。
        + `put()`：插入指定元素至队尾，由于是无界队列，所以会立即返回，无返回值。
        + `add()`：插入指定元素至队尾，由于是无界队列，所以会立即返回，返回值为true。
    + `SYNC`：同步操作，阻塞调用线程。
        + `transfer()`：阻塞直到出现一个消费者线程。
        + `take()`：从队首移除一个元素，如果队列为空，则阻塞线程。
    + `TIMED`：限时同步操作，限时阻塞调用线程。
        + `poll()`
        + `tryTransfer()`
    以上的操作通过4个常量传入调用`xfer()`。
    ```
    private static final int NOW   = 0; // for untimed poll, tryTransfer
    private static final int ASYNC = 1; // for offer, put, add
    private static final int SYNC  = 2; // for transfer, take
    private static final int TIMED = 3; // for timed poll, tryTransfer
    ```
    模拟入队出队过程分析一下这个方法，在代码中标出行数编号用于表示过程：
    + 第一次入队，队列为空时，使用`transfer()`方法。
        + → (1)判断头节点是否为空，头为空 
        + → (10)插入不为NOW 
        + → (11)插入新数据生成新节点 
        + → (12)第一个元素所以作为头节点 
        + → (14)阻塞该线程，返回（如果是ASYNC的话则 -> (15)插入成功）
    + 第二次入队
        + → (1)判断头节点是否为空，头不为空 
        + → (2)判断节点是否匹配过，没有匹配 
        + → (3)和头节点是同类型节点，匹配并跳出循环 
        + → (10)插入不为NOW 
        + → (11)插入新数据生成新节点 
        + → (12)追加到头节点后
        + → (14)阻塞该线程，返回
    + 第三次入队 和第二次入队一致。
    + 第一次出队，此时有三个线程在等待消费，使用`take()`。
        + → (1)判断头节点是否为空，头不为空
        + → (2)判断节点是否匹配过，第一个节点没有被匹配过因为匹配过的节点item为null 
        + → (4)设置头节点为null，如果没有其他线程竞争，设置成功 
        + → (5)因为头部已经被改为null了 p!=h 
        + → (8)尝试唤醒匹配节点的线程，出队，返回数据。
                唤醒的线程会把原本头节点值设置为自身（this），当前结点的等待线程为null。
    + 第二次出队：
        + → (1)判断头节点是否为空，头不为空
        + → (2)判断节点是否匹配过，head已经匹配过了 
        + → (9)找下一个节点，继续循环
        + → (1)(2)判断节点是否匹配过,head.next没有匹配过
        + → (4)设置节点item为null，如果没有其他线程竞争，设置成功 
        + → (5)此时p为head.next，q!=h
        + → (6)如果p.next不为空且head还没改变，修改头节点为p.next
        + → (8)尝试唤醒匹配节点的线程，出队，返回数据。
                唤醒的线程会把原本头节点值设置为自身（this），当前结点的等待线程为null。
    + 第三次出队，类似。           
            
    ```
    // haveData在入队时为true，出队时为false
    private E xfer(E e, boolean haveData, int how, long nanos) {
        // 不允许插入null值
        if (haveData && (e == null))
            throw new NullPointerException();
        Node s = null;                        // the node to append, if needed

        retry:
        for (;;) {                            // restart on append race
            // 匹配队列头部（第一个节点）
            // 如果是第一次插入，for循环在插入成功后才进入
            for (Node h = head, p = h; p != null;) { // find & match first node     // 1
                // 判断节点类型，取节点值
                boolean isData = p.isData;
                Object item = p.item;
                // 判断节点是否匹配匹配过
                if (item != p && (item != null) == isData) { // unmatched           // 2
                    // 如果是同个类型的节点就不能匹配
                    if (isData == haveData)   // can't match                        // 3
                        break;
                    // 当队列中已经有写操作在等待的时候，有读操作请求，把它匹配第一个写操作
                    // 设置头节点为null
                    if (p.casItem(item, e)) { // match                              // 4
                        // 
                        for (Node q = p; q != h;) {                                 // 5
                            Node n = q.next;  // update by 2 unless singleton   
                            // 一次性更新2步
                            // head -> p -> n 变成 p -> n(head)
                            // 原本head节点自引用等待回收
                            if (head == h && casHead(h, n == null ? q : n)) {       // 6
                                h.forgetNext();
                                break;
                            }                 // advance and retry
                            if ((h = head)   == null ||
                                (q = h.next) == null || !q.isMatched())             // 7
                                break;        // unless slack < 2
                        }
                        // 唤醒匹配的线程
                        LockSupport.unpark(p.waiter);                               // 8
                        // 出队，返回数据
                        return LinkedTransferQueue.<E>cast(item);
                    }
                }
                Node n = p.next;                                                    // 9
                p = (p != n) ? n : (h = head); // Use head if p offlist
            }
            // 判断是否为即使操作
            if (how != NOW) {                 // No matches available               // 10
                // 
                if (s == null)                                                      
                    s = new Node(e, haveData);                                      // 11
                // 尝试追加节点
                Node pred = tryAppend(s, haveData);                                 // 12
                if (pred == null)                                                   // 13
                    continue retry;           // lost race vs opposite mode
                // 判断是否为异步操作
                if (how != ASYNC)                                                   
                    return awaitMatch(s, pred, e, (how == TIMED), nanos);           // 14
            }
            return e; // not waiting                                                // 15
        }
    }    
    ```
___
###### DelayQueue
`DelayQueue<E extends Delayed>`的队列元素需要实现`Delayed`接口。
属于无界队列。
数据存储采用`PriorityQueue`（优先级队列），队列元素按照到期时间排序，而非入队顺序。
它提供了在指定时间才能获取队列元素的功能。
+ `Delayed`接口
队列元素需要实现`getDelay(TimeUnit unit)`方法和`compareTo(Delayed o)`方法, 
`getDelay`定义了剩余到期时间，
`compareTo`方法定义了元素排序规则，注意，元素的排序规则影响了元素的获取顺序。
+ 入队：不会造成阻塞。按照优先级排序。
+ 出队：如果生产者未到期，则需要消费者等待，直到生产者到期被消费者消费。
    如果生产者到期时，还没有消费者在等待，则该生产者延期等待。
    所以消费者线程的数量要够，处理任务的速度要快。
    否则，队列中的到期元素无法被及时取出并处理，造成任务延期、队列元素堆积等情况
+ 遍历：实现了`Iterator`接口，但遍历顺序不保证是元素的实际存放顺序。
___
#### ConcurrentSkipListMap
`ConcurrentSkipListMap`是线程安全的有序的哈希表。实现了`ConcurrentNavigableMap`接口，是基于**跳表**结构实现。
利用CAS操作实现非阻塞读/写/删除的有序key的map。它的思想为**用空间换时间**。

+ 结构
    
    `ConcurrentSkipListMap`采用`HeadIndex`内部类作存储结构，继承自内部类`Index`。
    整体跳表的结构为`HeadIndex`，包括了当前层数和每层的结构。
    ```
    static final class HeadIndex<K,V> extends Index<K,V> {
        final int level;
        HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
            super(node, down, right);
            this.level = level;
        }
    }    
    ```
    `Index`是每层跳表的结构，包含了节点和对应的下层Index。
    ```
    static class Index<K,V> {
        final Node<K,V> node;
        final Index<K,V> down;
        volatile Index<K,V> right;
        ...
    }
    ```

+ 初始化
    
    默认初始化一个自然排序的比较器并生成一个特殊标志的跳表头节点，层数为第一层，其他都为null。
    第一层包含了所有元素，上一层数据是下一层数据的子集。层次越高，跳跃性越大，包含的数据越少。
    head -> HeadIndex(level:1) -> Node(value: BASE_HEADER)
    
+ 插入

    插入主要调用`doPut()`方法：
    主要简单可以分为几个步骤：
    + 找到新增元素key的前继节点
    + 从第一层Index开始遍历，如果key > n.key 继续向右，如果key == n.key，那就替换，否则向下
    + 逐层以此规律遍历直到最底层插入。
    + 如果过程中需要增加索引层，搞个HeadIndex往下链接到Index直到最下层插入数据。
    
    ```doPut
    private V (K key, V value, boolean onlyIfAbsent) {
        Node<K,V> z;             // added node
        if (key == null)
            throw new NullPointerException();
        Comparator<? super K> cmp = comparator;
        outer: for (;;) {
            // 找key对应的前继结点b，b的next节点n
            for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
                // 判断n是否为空，如果n为空，跳过if语句
                // 如果n不为空
                if (n != null) {
                    Object v; int c;
                    Node<K,V> f = n.next;
                    // 如果存在条件竞争，重新开始循环
                    if (n != b.next)               // inconsistent read
                        break;
                    // 如果n已经被删除了，调用helpDelete帮忙删除，重新开始循环
                    if ((v = n.value) == null) {   // n is deleted
                        n.helpDelete(b, f);
                        break;
                    }
                    // 值为null，说明b已经被删除了
                    if (b.value == null || v == n) // b is deleted
                        break;
                    // 如果key大于n.key则进入if语句，继续往后遍历
                    if ((c = cpr(cmp, key, n.key)) > 0) {
                        b = n;
                        n = f;
                        continue;
                    }
                    // key和n.key相等，直接赋值，采用CAS更新
                    if (c == 0) {
                        if (onlyIfAbsent || n.casValue(v, value)) {
                            @SuppressWarnings("unchecked") V vv = (V)v;
                            return vv;
                        }
                        // 竞争失败，重新来过
                        break; // restart if lost race to replace value
                    }
                    // else c < 0; fall through
                }

                z = new Node<K,V>(key, value, n);
                // 把新节点插入b.next
                if (!b.casNext(n, z))
                    break;         // restart if lost race to append to b
                break outer;
            }
        }
        // 生成一个随机数种子
        int rnd = ThreadLocalRandom.nextSecondarySeed();
        // if为true时需要增加层数
        if ((rnd & 0x80000001) == 0) { // test highest and lowest bits
            int level = 1, max;
            while (((rnd >>>= 1) & 1) != 0)
                ++level;
            // 上面获取到的level为新的层数    
            Index<K,V> idx = null;
            HeadIndex<K,V> h = head;
            // 新层数没有超过最大值（head的level为最大层值）
            if (level <= (max = h.level)) {
                // 链接level个垂直的Index，idx最终指向最高层的Index节点
                for (int i = 1; i <= level; ++i)
                    idx = new Index<K,V>(z, idx, null);
            }
            // 新层数超过了最大层数
            else { // try to grow by one level
                // 新层数 = 最大层数 + 1，重置level
                level = max + 1; // hold in array and later pick the one to use
                // 生成一个Index结点数组,idxs[0]不会使用
                @SuppressWarnings("unchecked")Index<K,V>[] idxs =
                    (Index<K,V>[])new Index<?,?>[level+1];
                for (int i = 1; i <= level; ++i)
                    idxs[i] = idx = new Index<K,V>(z, idx, null);
                // 生成新的HeadIndex结点    
                for (;;) {
                    h = head;
                    int oldLevel = h.level;     // 原最大层数
                    // 有其他线程对index层进行增加操作，不做了
                    if (level <= oldLevel) // lost race to add level
                        break;
                    HeadIndex<K,V> newh = h;
                    // 这里的 oldbase 就是BASE_HEADER
                    Node<K,V> oldbase = h.node; 
                    // 增加一层HeadIndex，把新的头链接到原来的头上，形成一条索引链
                    for (int j = oldLevel+1; j <= level; ++j)
                        newh = new HeadIndex<K,V>(oldbase, newh, idxs[j], j);
                    // 再CAS更新头节点，形成新的headIndex
                    if (casHead(h, newh)) {
                        h = newh;
                        // 这里的 idx 上从上往下第二层的 index 节点 level 也变成的 第二
                        idx = idxs[level = oldLevel];
                        break;
                    }
                }
            }
            // 链接各层的各个HeadIndex和Index节点
            // find insertion points and splice in
            splice: for (int insertionLevel = level;;) {
                int j = h.level;
                for (Index<K,V> q = h, r = q.right, t = idx;;) {
                    // 节点都被删除了，那就退出吧
                    if (q == null || t == null)
                        break splice;
                    if (r != null) {
                        // 向右遍历，和右边的节点对比，如果当前key小于右边的key，那就继续向右
                        Node<K,V> n = r.node;
                        // compare before deletion check avoids needing recheck
                        int c = cpr(cmp, key, n.key);
                        // 帮助删除
                        if (n.value == null) {
                            if (!q.unlink(r))
                                break;
                            r = q.right;
                            continue;
                        }
                        if (c > 0) {
                            q = r;
                            r = r.right;
                            continue;
                        }
                    }
                    // 到这证明 key < n.key
                    if (j == insertionLevel) {
                        if (!q.link(r, t))
                            // 插入竞争失败
                            break; // restart
                        if (t.node.value == null) {
                            // node已经被删除了
                            // 通过findPredecessor清理index层, findNode清理node层
                            findNode(key);
                            break splice;
                        }
                        // 当前第index层添加完成，为下一层index做准备
                        if (--insertionLevel == 0)
                            break splice;
                    }

                    // 简单理解：当前层的index都添加完成了，那就做下一层
                    if (--j >= insertionLevel && j < level)
                        t = t.down;
                    q = q.down;     // 往下查
                    r = q.right;
                }
            }
        }
        return null;
    }
    ```      
+ 删除：`doRemove()`方法，在删除时，如果是非索引节点，则在待删除节点和next节点中间插入一个null标记节点，
    然后改变next节点的前驱节点，使待删除节点和标记节点一起等待垃圾回收。
    如果是索引节点，在上面的过程之后会调用findPredecessor去接触待删除节点上的index节点的引用，最后一起被清理。
+ 查找：从左往右，从上到下，逐步找到对应key。
+ 问题：
___

+ [返回顶部](#目录)