# 集合与JUC笔记
## 基础集合扫盲
1. Collection

    是所有集合的基类（接口）。
    Java 8支持lambda语法。新增了不少的方法。（待拓展）
    1. **List**
    
        List是一个有序可重复的集合，允许有null值。
        
        + List提供了一个特殊的迭代器`ListIterator`。
        + 提供了`add()`, `remove(Object o)`, `addAll()`, `removeAll()`, `clear()`的新增删除操作方法。
        + 提供了`contains()`, `containsAll()`的对比方法。
        + 提供了`get()`, `set()`的查询方法
        
        **Iterator** 迭代器为集合提供的迭代器，只能单向移动，用于遍历集合。`Iterator.remove()`是**唯一安全**的方式来在迭代过程中修改集合。
            
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
                + `forEach(Consumer<? super E> action)`
                    
                    (涉及Lambda)
                
                + `spliterator()`
                
                    Spliterator 为并行遍历数据源中的元素的迭代器，基于索引的、二分的、懒加载器。
                    (涉及Java Streams和Lambda)
            
        2. **LinkedList**
            
            双向链表。实现`List`接口, `Deque（双端队列）`接口。非线程安全，在多线程环境下操作需进行同步。
            特点：新增、删除效率高。遍历时，为从头逐个遍历，效率低。
            
            + 经典问题：ArrayList和LinkedList的区别
        
        3. Vector
        
            类似ArrayList，线程安全。实现了List接口。大部分方法都进行了同步。已弃用。

        4. **Collections.synchronizedList(List list)**
        
            Collections提供了将集合转为线程安全的集合。Collections内置`SynchronizedList`类实现 List 接口方法时，内部都加上了 synchronized 代码块。
            但 Iterator 没有实现同步，所以在使用时需要对 Iterator 进行同步。
     
    2. **Map**
        
        Key-Value键值对（一对一）数据结构。Key是唯一的。
        + 提供了`put()`, `remove()`, `putAll()`, `removeAll()`的新增删除方法。
        + 提供了`keySet()`, `values()`, `entrySet()`获取键、值集合的方法。
        
        1. **HashMap**
            实现了`Map`接口，允许key和value为null。非线程安全。
            + 默认初始容量为 16，负载因子为 0.75。
            + 默认最大容量为 `1<<30（2^30）`。超过默认最大容量后的最大容量为`Integer.MAX_VALUE`。
                + 刁钻问题：为什么是 30 而不是 31 呢？
            + 采用了**数组 + 链表**的结构
                
            + **扩容机制**
            
                当map中填满了 负载因子*容器容量 数量的时候，map将进行扩容，扩容后数组的大小为原来HashMap大小的两倍。
                比如容量为16，负载因子为0.75，则在数组装满12（16*0.75=12）个元素的时候进行扩容，扩容后数组大小为32。
                map扩容后，对所有元素重新hash计算，再形成新的map。
                
            + 哈希碰撞
            
                哈希计算公式：`hash = key.hashcode & (length - 1)`。当哈希值相同时，遵循先来后到的原则，以链表的形式链接。
                
                注：`&`运算规则为两个数以二进制的形式进行比较，两个位都是1，结果才是1。
            + **线程不安全**
            
                因为HashMap的扩容机制，存在可能多个线程对一个HashMap进行操作。
                
                当多个线程对同个HashMap在进行插入操作并且HashMap需要进行扩容时，存在出现循环引用的问题。
            
        2. **TreeMap**
        
        3. **Hashtable**
        
        4. **Collections.synchronizedMap(Map map)**
            
            Collections类提供了将Map转为线程安全的Map集合。Collections内置了`SynchronizedMap`类在进行操作的时候添加了synchronized同步代码块。
            同样，Iterator 没有实现同步，所以在使用时需要对 Iterator 进行同步。
            
            __**建议使用ConcurrentHashMap类。**__
            
    3. **Set**
        
        无序不重复的集合。底层数据结构使用 Map 实现。将Map的Key作为Set集合的内容，保证了唯一性。
        
        + HashSet: 底层使用 HashMap。
        + TreeSet: 自然排序的Set，底层使用 TreeMap。加入的元素必须实现 Comparable 接口。

## JUC(java.util.concurrent)扫盲
