# 建造者模式

建造者模式使用多个简单的对象创建复杂的对象，用于将一个复杂的构建与其表示分离，使用同样的构建过程可以创建不同的表示，然后通过一个Builder类（该Builder类是独立于其他对象的）创建最终的对象。

**建造者模式主要用于解决软件系统中复杂对象的创建问题，比如有些复杂对象的创建需要通过各部分的字对象用一定的算法构成，**在需求变化时这些复杂对象将面临很大的改变，这十分不利于系统的稳定。

但是，使用建造者模式能将他们各部分的算法包装起来，在需求变化后只需调整各个算法的组合方式和顺序，能极大提高系统的稳定性。建造者模式常被用于一些基本部件不回变化而其组合经常变化的应用场景下。

tips：建造者模式与工厂模式的最大区别是，建造者模式更关注产品的组合方式和装配顺序，而工厂模式关注产品的生产本身。

#### 建造者模式在设计时的几个角色

* ##### Builder：创建一个复杂产品对象的抽象接口。

* ##### ConcreteBuilder：Builder接口的实现类，用于定义复杂产品各部件的装配流程。

* ##### Director：构造一个使用Builder接口的对象。

* ##### Product：表示构造的复杂对象。ConcreteBuilder定义了该复杂对象的装配流程，而Product定义了该复杂对象的接口和内部表示。

#### 示例

* 类图

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/建造者模式.png)

* 代码

##### 1. 创建ComputerBuilder接口

```java
public interface ComputerBuilder {

    void builderCpu();

    void builderMerory();

    void builderDisk();

    Computer builderComputer();
}
```

##### 2. 创建实现类ComputerConcreteBuilder

```java
public class ComputerConcreteBuilder implements ComputerBuilder{

    private Computer computer;

    public ComputerConcreteBuilder(Computer computer) {
        this.computer = computer;
    }

    @Override
    public void builderCpu() {
        System.out.println("构造电脑cpu");
        this.computer.setCpu("苹果M1");
    }

    @Override
    public void builderMerory() {
        System.out.println("构造电脑内存");
        this.computer.setMerory("DRR4");
    }

    @Override
    public void builderDisk() {
        System.out.println("构造电脑硬盘");
        this.computer.setDisk("Disk");
    }

    @Override
    public Computer builderComputer() {
        return computer;
    }
}
```

##### 3. 创建构造一个使用Builder接口的对象 ComputerDirector

```java
public class ComputerDirector {

    public Computer constructComputer(ComputerBuilder computerBuilder) {
        computerBuilder.builderMerory();
        computerBuilder.builderCpu();
        computerBuilder.builderDisk();
        return computerBuilder.builderComputer();
    }
}
```

##### 4. 测试

```java
public class MainTest {

    public static void main(String[] args) {

        Computer computer = new Computer();
        ComputerBuilder computerBuilder = new ComputerConcreteBuilder(computer);
        ComputerDirector computerDirector = new ComputerDirector();
        computerDirector.constructComputer(computerBuilder);
    }
}

```

##### 5. Computer实体类

```java
@Setter
@Getter
@ToString
public class Computer {

    private String cpu;
    private String merory;
    private String disk;
}

```



