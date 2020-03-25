# 集合与JUC笔记

## 目录

- [基础集合扫盲](#基础集合扫盲)
    - [Collection](#Collection)
        - [List](#List)
        - [Map](#Map)
        - [Set](#Set)
- [JUC(java.util.concurrent)扫盲](#JUC扫盲)
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

`java.util.concurrent`包 简称 JUC 包。JUC离不开 **CAS(Compare And Swap)**和**volatile**。

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
由于`Hashtable`在操作时使用`synchronized`对整个对象进行加锁（锁住了整个Hash表），导致`ashtable`效率低下。而`HashMap`为非线程安全。
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
        
        如果当前节点数binCount超过8个，则进行数组扩容或把节点转为树。
        最后使用`addCount()`进行累积当前节点的数量，并判断是否需要扩容。
        
        `put`调用`putVal`的代码：
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
                // 需要扩容，调用transfer()方法
                ……
            }
        }        
        ```        
    + 扩容机制：
         
        扩容包括了**容量变大**和**数据迁移到新集合内**。
        这里的扩容指初始化后对数组容量进行扩大。
        还有协助扩容复制
      
        ```
        // （待分析）
        // 扩容方法，大小总是 2^n
        // 传入的size是原来数组长度的2倍
        private final void tryPresize(int size) {
            // 如果size大于 MAXIMUM_CAPACITY（1<<30） 最大容量的一半，则用最大容量，
            // 否则使用tableSizeFor算出来
            int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
                tableSizeFor(size + (size >>> 1) + 1);
            int sc;
            while ((sc = sizeCtl) >= 0) {
                Node<K,V>[] tab = table; int n;
                // 判断是否初始化
                if (tab == null || (n = tab.length) == 0) {
                    // 初始化一个大小为sizeCtrl和刚刚算出来的c中较大的一个大小的数组
                    n = (sc > c) ? sc : c;
                    // CAS更新，获取锁
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
                // 一直扩容到c <= sizeCtl或者数组长度大于最大长度的时候就退出
                // 一次扩容是原来长度的2^n倍
                else if (c <= sc || n >= MAXIMUM_CAPACITY)
                    break;
                else if (tab == table) {
                    int rs = resizeStamp(n);
                    if (sc < 0) {
                        Node<K,V>[] nt;
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
                }
            }
        }    
        ```
        ```
        // 帮助转移数据（待分析）
        final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
            Node<K,V>[] nextTab; int sc;
            if (tab != null && (f instanceof ForwardingNode) &&
                (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
                int rs = resizeStamp(tab.length);
                while (nextTab == nextTable && table == tab &&
                       (sc = sizeCtl) < 0) {
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
    
        
    
___
#### CopyOnWriteArrayList

___
#### ConcurrentLinkedQueue

___
#### BlockingQueue

___
#### ConcurrentSkipListMap

___
