# 集合及 MAP

## ArrayList 和 LinkedList 区别和优缺点
+ ArrayList 是动态数组，是基于索引(index)的数据结构。
查询快，增删慢。
+ LinkedList 是双向链表。
查询慢，增删快。

## 在基础集合里面，哪些线程安全
基础 Collection 集合类中，Vector 和 Hashtable 为线程安全的集合。
+ Vector：与 ArrayList 一样，通过数组实现。它支持线程的同步。访问较慢。底层使用`synchronized`实现。已弃用。
+ Hashtable：与 HashMap 相似。几乎所有public方法都加了`synchronized`。因性能问题也被弃用了。

## Hashtable和 ConcurrentHashMap 的区别（未完善）
Hashtable 和 ConcurrentHashMap 都为线程安全的集合。
+ 锁机制不同。
    + Hashtable使用的是public方法加上`synchronized`关键字，一次锁住整个hash表。
    + ConcurrentHashMap采用了 CAS + synchronized 来保证并发安全性。

