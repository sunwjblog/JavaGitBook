# 单例模式

单例模式是保证系统实例唯一性的重要手段，单利模式首先通过讲勒的实例化方法私有化来防止程序通过其他方式创建该类的实例，然后通过提供一个全局唯一获取该类实例的方法帮助用户获取类的实例，用户只需也只能通过调用该方法获取类的实例。

单例模式的设计保证了一个类在整个系统中同一时刻只有一个实例存在，主要被用于一个全局类的对象在多个地方被使用并且对象的状态是全局变化的场景下。

单利模式的常见写法有懒汉模式（线程安全）、饿汉模式、静态内部类、双重校验锁。

### 示例

* ##### 懒汉模式代码

```java
public class LazySingletonPattern {
    
    private static LazySingletonPattern singletonPattern = null;
    
    private LazySingletonPattern(){}
    
    public static synchronized LazySingletonPattern getInstance() {
        if (singletonPattern == null)
            singletonPattern = new LazySingletonPattern();
        
        return singletonPattern;
    }
}
```

* ##### 饿汉模式代码

```java
public class HungrySingletonPattern {

    private static HungrySingletonPattern hungrySingletonPattern = new HungrySingletonPattern();

    private HungrySingletonPattern() {}

    public static HungrySingletonPattern getInstance() {
        return hungrySingletonPattern;
    }
}
```

* ##### 静态内部类

静态内部类通过在类中定义一个静态内部类，将对象示例的定义和初始化放在内部类中完成，我们在获取对象时要经过静态内部类调用其单例对象。

之所以这样设计，是应为类的静态内部类是JVM中唯一的，这也很好地保证了单例对象的唯一性。

```java
public class SingletonInnerClass {

    private static class SingletonHolder {
        private static final SingletonInnerClass INSTANCE = new SingletonInnerClass();
    }

    private SingletonInnerClass() {}

    public static SingletonInnerClass getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

* ##### 双重检验锁

双锁模式指在懒汉模式的基础上做进一步优化，给静态对象的定义加上volatile锁来保障初始化时对象的唯一性，在获取对象时通过synchronized(Singleton.class)给单例类加锁来保障操作的唯一性。

```java
public class Lock2Singleton {

    private volatile static Lock2Singleton singleton;

    private Lock2Singleton() {}

    public static Lock2Singleton getInstance() {

        if (singleton == null) {
            synchronized (SingletonInnerClass.class) {
                if (singleton == null)
                    singleton = new Lock2Singleton();
            }
        }
        return singleton;
    }
}
```



