# 基础题目

## == 和 equals 的区别
==是判断两个变量或实例是不是指向同一个内存空间，指对内存地址进行比较，引用是否相同。

equals是判断两个变量或实例所指向的内存空间的值是不是相同，对字符串的内容进行比较，值是否相同。

包装类中，Byte、Short、Integer、Long、Character  均有缓存数据。

+ Byte、Short、Integer、Long 缓存范围是：[-128, 127]。
+ Character 缓存范围：[0, 127]。
+ Float、Double 无缓存。

在有运算表达式参与比较时，均为值比较。

例子：

    int a = 1;
    int b = 2;
    Integer c = 1;      // Integer c = 1; 等同于 Integer c = Integer.valueOf(1);
    Integer d = 2;
    Integer e = new Integer(2);
    Long f = 1L;
    Long g = new Long(2L);
    Integer h = 321;
    Integer i = new Integer(321);
    System.out.println(a == c);         // true，值比较
    System.out.println(b == e);         // true，值比较
    System.out.println(d == e);         // false，对象比较
    System.out.println(d.equals(e));    // true，缓存
    System.out.println(a == f);         // true，值比较
    System.out.println(g == (a+c));     // true，值比较
    System.out.println(g.equals(a+c));  // false，值比较
    System.out.println((a+b) == (c+d)); // true，值比较
    System.out.println(h == i);         // false，对象比较
    System.out.println(h.equals(i));    // true，值比较
   

## 