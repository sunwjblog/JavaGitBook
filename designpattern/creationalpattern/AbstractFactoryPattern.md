# 抽象工厂模式

抽象工厂模式在工厂模式添加了一个创建不同工厂的抽象接口（抽象类或接口实现），该接口可叫超级工厂。

我们可以将工厂模式理解为针对一个产品纬度进行分类，比如在工厂模式下的苹果手机和华为手机；而抽象工厂模式针对的是多个产品纬度分类，比如苹果公司既制造苹果手机也制造苹果笔记本电脑，同样的，华为公司也是如此。

在同一个厂商有多个维度的产品时，如果使用工厂模式，则势必会存在多个独立的工厂，这样的话，设计和无力世界时不对应的。正确的做法是通过抽象工厂模式来实现，我们可以将抽象工厂类比成厂商（苹果，华为），将通过抽象工厂创建出来的工厂类比坐不同产品的生产线（手机生产线、笔记本生产线），在需要生产产品时根据抽象工厂生产。

类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/抽象工厂模式类图.png)

### 代码

* ##### 抽象工厂类 AbstractFactory

```java
public abstract class AbstractFactory {

    public abstract Phone createPhone(String brand);

    public abstract Computer createComputer(String brand);
}

```

* ##### 创建两条产品线 ComputerFactory 和 PhoneFactory

```java
public class ComputerFactory extends AbstractFactory {

    @Override
    public Phone createPhone(String brand) {
        return null;
    }

    @Override
    public Computer createComputer(String brand) {
        if ("HuaWei".equals(brand))
            return new HuaWeiComputer();
        else if ("Apple".equals(brand))
            return new ComputerApple();
        else
            return null;
    }
}

```

```java
public class PhoneFactory extends AbstractFactory {

    @Override
    public Phone createPhone(String brand) {

        if ("HuaWei".equals(brand))
            return new HuaWeiPhone();
        else if ("Apple".equals(brand))
            return new ApplePhone();
        else
            return null;
    }

    @Override
    public Computer createComputer(String brand) {
        return null;
    }
}
```

* ##### 创建生产产品的接口 Computer 和 Phone

```java
public interface Computer {

    String intenmet();
}
```

```java
public interface Phone {

    String call();
}
```

* ##### 不同产品的实例

```java
public class HuaWeiPhone implements Phone{
    @Override
    public String call() {
        return "this is a HuaWei Phone";
    }
}

public class ApplePhone implements Phone{
    @Override
    public String call() {
        return "this is a Apple Phone";
    }
}

```

```java
public class ComputerApple implements Computer{
    @Override
    public String intenmet() {
        return "this is a Apple Computer";
    }
}

public class HuaWeiComputer implements Computer{
    @Override
    public String intenmet() {
        return "this is a HuaWei Computer";
    }
}

```

* 测试代码

```java
public class MainTest {

    public static void main(String[] args) {

        AbstractFactory phoneFactory = new PhoneFactory();
        Phone applePhone = phoneFactory.createPhone("Apple");
        Phone huaWeiPhone = phoneFactory.createPhone("HuaWei");
        System.out.println(applePhone.call());
        System.out.println(huaWeiPhone.call());

        AbstractFactory computerFactory = new ComputerFactory();
        Computer appleComputer = computerFactory.createComputer("Apple");
        Computer huaweiComputer = computerFactory.createComputer("HuaWei");
        System.out.println(appleComputer.intenmet());
        System.out.println(huaweiComputer.intenmet());
    }
}
```

