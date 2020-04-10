# 并发

## synchronized 底层实现
synchronized底层是通过监视锁（monitor）实现的。使用synchronized时，JVM会根据当前使用环境找到对应对象的monitor，再根据monitor的状态进行加、解锁的判断。
+ synchronized代码块

源码：

    
    public class TestSynchronized {
        public void method() {
            synchronized (this) {
                System.out.println("Method start!");
            }
        }
    }
    
编译结果：

    public class com.demo.juc.TestSynchronized { 
        public com.demo.juc.TestSynchronized();
            Code:
                0: aload_0
                1: invokespecial #1                  // Method java/lang/Object."<init>":()V
                4: return        
                
        public void method1();
            flags: ACC_PUBLIC
            Code:
            0: aload_0 
            1: dup         
            2: astore_1     
            3: monitorenter                      // **Point 1**
            4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
            7: ldc           #3                  // String Method start!  
            9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
            12: aload_1
            13: monitorexit                      // **Point 2**
            14: goto          22   
            17: astore_2
            18: aload_1  
            19: monitorexit   
            20: aload_2     
            21: athrow      
            22: return   
        Exception table: 
            from    to  target  type  
            4       14    17    any        
            17      20    17    any    
            
        public synchronized void method2();
            flags: ACC_PUBLIC, ACC_SYNCHRONIZED          // **Point 3**
            Code:
                    0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
                    3: ldc           #5                  // String Method2 start!
                    5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
                    8: return
    }  

+ 结果简要分析
    + 同步代码块中，使用`monitorenter`(Point 1)和`monitorexit`(Point 2)两个字节码指令获取和释放monitor。
    + 同步方法中，使用`ACC_SYNCHRONIZED`(Point 3)标识该方法为同步方法。

## synchronized 锁过程(待确认)
底层实现过程
+ monitorenter 

每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：
1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。
2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.
3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

+ monitorexit

执行monitorexit的线程必须是objectref所对应的monitor的所有者。
指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。 

## 线程池初始化过程？


## 线程池中的队列使用的是哪种？
（自答）
线程池中用到的队列为阻塞队列，分别有 有界队列`ArrayBlockingQueue`, 无界队列`LinkedBlockingQueue`,
优先级无界队列`PriorityBlockingQueue`, 特殊无元素存储空间队列`SynchronousQueue`四种阻塞队列。
常用的为无界队列`LinkedBlockingQueue`，因为它是边界可选队列，可以当无界队列也可以当有界队列。

## 简述一下volatile


## 乐观锁和悲观锁
+ 乐观锁

    总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和CAS算法实现。乐观锁适用于多读的应用类型，这样可以提高吞吐量
+ 悲观锁

    总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现

## CAS


## 简述一下ConcurrentHashMap在JDK7和JDK8的区别，以及算法原理。



## 设计2个线程，一个线程打印奇数（1,3,5,7,……,97,99），一个线程打印（2,4,6,8,……,98,100），并且2个线程并发运行结果为自然排序输出（1,2,3,4,……,98,99,100）
