# 原型模式

**原型模式指通过调用原形实例的Clone方法或其他手段来创建对象。**

原型模式的Java实现很简单，只需原型类实现Cloneable接口并覆写clone方法即可。

#### 深复制和浅复制

Java中的复制分为浅复制和深复制

* 浅复制：Java中的浅复制是通过实现Cloneable接口并覆写其Clone方法实现的。在浅复制的过程中，对象的基本数据类型的变量值会重新被复制和创建，而引用数据类型仍指向原对象的引用。也就是说，浅复制不复制对象的引用类型数据。
* 深复制：在深复制的过程中，不论是基本数据类型还是引用数据类型，都会被重新复制和创建。简而言之，深复制彻底复制了对象的数据（包括基本数据类型和引用数据类型），浅复制的复制却并不彻底（忽略了引用数据类型）。

#### 示例代码

* ##### 浅复制

```java
@ToString
public class Computer implements Cloneable {

    private String cpu;
    private String memory;
    private String disk;

    public Computer(String cpu, String memory, String disk) {
        this.cpu = cpu;
        this.memory = memory;
        this.disk = disk;
    }

    public Object clone() {
        try {
            return (Computer)super.clone();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```



* 深复制

```java
@ToString
public class ComputerDetail implements Cloneable {

    private String cpu;
    private String memory;
    private Disk disk;

    public ComputerDetail(String cpu, String memory, Disk disk) {
        this.cpu = cpu;
        this.memory = memory;
        this.disk = disk;
    }

    public Object clone() {
        try {
            return (ComputerDetail)super.clone();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

```java
@ToString
public class Disk implements Cloneable {

    private String ssd;
    private String hhd;

    public Disk(String ssd, String hhd) {
        this.ssd = ssd;
        this.hhd = hhd;
    }

    public Object clone() {
        try {
            return (Disk)super.clone();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

* 测试

```java
public class MainTest {

    public static void main(String[] args) {

        // 浅复制
        Computer computer = new Computer("8core", "16G", "1TB");
        System.out.println("浅复制 computer before>>" + computer);
        Computer computerClone = (Computer) computer.clone();
        System.out.println("浅复制 computer after>>" + computerClone);

        // 深复制
        Disk disk = new Disk("208G", "2TB");
        ComputerDetail computerDetail = new ComputerDetail("8core", "16G", disk);
        System.out.println("深复制 computerDetail before>>" + computerDetail);
        ComputerDetail computerDetailClone = (ComputerDetail) computerDetail.clone();
        System.out.println("浅复制 computerDetail after>>" + computerDetailClone);
    }
}
```

