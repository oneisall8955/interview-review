# 设计模式

## 单例
单例模式，指一个类只有一个实例，且该类能自行创建这个实例的一种模式。
分为**饿汉式单例**和**懒汉式单例**。
+ 懒汉式单例

该模式的特点是类加载时没有生成单例，只有当第一次调用`getlnstance`方法时才去创建这个单例。

    public class LazySingleton {
        private static volatile LazySingleton instance=null;    //保证 instance 在所有线程中同步
        private LazySingleton(){}    //private 避免类在外部被实例化
        public static synchronized LazySingleton getInstance() {
            //getInstance 方法前加同步
            if(instance==null)
            {
                instance=new LazySingleton();
            }
            return instance;
        }
    }
    
多线程下因为`synchronized`会影响性能。
    
+ 饿汉式单例

该模式的特点是类一旦加载就创建一个单例，保证在调用`getInstance`方法之前单例已经存在了。 


    public class HungrySingleton{
        private static final HungrySingleton instance=new HungrySingleton();
        private HungrySingleton(){}
        public static HungrySingleton getInstance() {
            return instance;
        }
    }
    
饿汉式单例在类创建的同时就已经创建好一个静态的对象供系统使用，以后不再改变，所以是线程安全的，可以直接用于多线程而不会出现问题。  

## 简述常用的几个设计模式
+ 单例模式：指一个类只有一个实例，且该类能自行创建这个实例的一种模式。
+ 工厂方法模式：定义一个创建产品对象的工厂接口，将产品对象的实际创建工作推迟到具体子工厂类当中。
+ 代理模式：为某对象提供一种代理以控制对该对象的访问。即客户端通过代理间接地访问该对象，从而限制、增强或修改该对象的一些特性。
+ 观察者模式：多个对象间存在一对多关系，当一个对象发生改变时，把这种改变通知给其他多个对象，从而影响其他对象的行为。
