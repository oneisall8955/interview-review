# Redis相关

## Redis结构

## Redis为什么是单线程而查询快

## Redis数据结构
+ String

    String数据结构是简单的key-value类型，value其实不仅可以是String，也可以是数字。
+ List

    list 就是链表，Redis list 的应用场景非常多，也是Redis最重要的数据结构之一

+ Hash

    hash 是一个 String 类型的 field 和 value 的映射表，hash 特别适合用于存储对象，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。
+ Set

    Set 对外提供的功能与list类似是一个列表的功能，特殊之处在于 set 是可以自动排重的。
+ ZSet（Sorted Set）

    和set相比，sorted set增加了一个权重参数score，使得集合中的元素能够按score进行有序排列。
