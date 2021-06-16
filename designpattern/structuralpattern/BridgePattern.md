# 桥接模式

桥接模式通过姜抽象机器实现接耦，使二者可以根据需求独立变化。这种类型的设计模式属于结构型模式，通过定义一个抽象和实现之间的桥接者来达到结构的目的。

桥接模型主要用于解决在需求多变的情况下使用继承造成类爆炸的问题，扩展起来不够灵活。可以通过桥接模式将抽象部分与实现部分分离，使其能够独立变化而相互之间的功能不受影响。

具体做法是通过定义一个桥接接口，使得实体类的功能独立于接口实现类，降低它们之间的耦合度。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/桥接模式类图.png)

1. #### 定义Driver接口

```java
public interface Driver {

    void executeSQL();
}

```

2. #### 定义实现Driver接口的实现类MysqlDriver和OracleDriver

```java
public class MysqlDriver implements Driver {

    @Override
    public void executeSQL() {
        System.out.println("execute sql by Mysql Driver.");
    }
}

```

```java
public class OracleDriver implements Driver{
    @Override
    public void executeSQL() {
        System.out.println("execute sql by Oracle Driver.");
    }
}
```

3. #### 定义DriverManagerBridge

```java
public class DriverManangerBridge {

    private Driver driver;

    public Driver getDriver() {
        return driver;
    }

    public void setDriver(Driver driver) {
        this.driver = driver;
    }

    public void executeSQL() {
        this.driver.executeSQL();
    }
}
```

4. #### 定义MyDriverBridge用于实现用户自定义的功能

```java
public class MyDriverBridge extends DriverManangerBridge{

    public void execute() {
        getDriver().executeSQL();
    }
}
```

5. #### 测试类

```java
public class BridgeTest {

    public static void main(String[] args) {

        DriverManangerBridge driverManangerBridge = new DriverManangerBridge();

        MysqlDriver mysqlDriver = new MysqlDriver();
        driverManangerBridge.setDriver(mysqlDriver);
        driverManangerBridge.executeSQL();


        OracleDriver oracleDriver = new OracleDriver();
        driverManangerBridge.setDriver(oracleDriver);
        driverManangerBridge.executeSQL();
    }
}
```

