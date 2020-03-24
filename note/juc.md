# 集合与JUC笔记
## 基础集合扫盲
### Collection

Collection是所有集合的基类（接口）。
Java 8支持lambda语法。新增了不少的方法。（待拓展）
___
#### **List**

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
#### **Map**

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
#### **Set**

无序不重复的集合。底层数据结构使用 Map 实现。将Map的Key作为Set集合的内容，保证了唯一性。
        
+ HashSet: 底层使用 HashMap。
+ TreeSet: 自然排序的Set，底层使用 TreeMap。加入的元素必须实现 `Comparable` 接口。
+ `Collections.synchronizedSet(Set set)`与前面一样。
___
## JUC(java.util.concurrent)扫盲

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
#### 解决高并发性能问题——LongAdder/DoubleAdder

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
### 并发容器

___
#### ConcurrentHashMap

___
#### CopyOnWriteArrayList

___
#### ConcurrentLinkedQueue

___
#### BlockingQueue

___
#### ConcurrentLinkedQueue

___
#### ConcurrentSkipListMap

___
