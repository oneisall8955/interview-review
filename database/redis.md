# Redis相关

## Redis线程模型
redis 内部使用文件事件处理器`file event handler`。这个文件事件处理器是单线程的，所以 redis 才叫做单线程的模型。
它采用 IO 多路复用机制同时监听多个 socket，根据 socket 上的事件来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

+ 多个 socket
+ IO 多路复用程序
+ 文件事件分派器
+ 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，
会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。

参考地址：https://www.javazhiyin.com/22943.html 
 
## Redis为什么是单线程而查询快
+ 纯内存操作
+ 核心是基于非阻塞的 IO 多路复用机制
+ 单线程反而避免了多线程的频繁上下文切换问题


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

## Redis持久化方式
Redis的一种持久化方式叫 **快照**（snapshotting，RDB），另一种方式是**只追加文件**（append-only file,AOF）。
+ 快照

    通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。Redis创建快照之后，可以对快照进行备份，
    可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis主从结构，主要用来提高Redis性能），
    还可以将快照留在原地以便重启服务器的时候使用。

快照持久化是Redis默认采用的持久化方式，在redis.conf配置文件中默认有此下配置：

    save 900 1           #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
    save 300 10          #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
    save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

+ AOF（append-only file）持久化

    默认情况下Redis没有开启AOF（append only file）方式的持久化，可以通过appendonly参数开启。
    
    
    appendonly yes

开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件。
AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的，默认的文件名是appendonly.aof。

在Redis的配置文件中存在三种不同的 AOF 持久化方式，它们分别是：

    appendfsync always    #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
    appendfsync everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘
    appendfsync no        #让操作系统决定何时进行同步

为了兼顾数据和写入性能，用户可以考虑 appendfsync everysec选项 ，让Redis每秒同步一次AOF文件，Redis性能几乎没受到任何影响。

