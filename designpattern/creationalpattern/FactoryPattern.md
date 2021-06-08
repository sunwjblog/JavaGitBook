# 工厂模式

工厂模式是最常见的设计模式，概模式属于创建型模式，它提供了一种简单、快速、高效而安全地创建对象的方式。

工厂模式在接口中定义了创建对象方法，而将具体的创建对象的过程在子类中实现，用户只需通过接口创建需要的对象即可，不用关注对象的具体创建过程。

工厂模式的本质就是用工厂方法代替new操作创建一种实例化对象的方式，以提供一种方便地创建有同样类型接口的产品的复杂对象。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/工厂模式类图.png)

### 代码实现

#### 接口Phone

```java
public interface Phone{
  String brand();
}
```

#### 实现类Iphone和HuaWei

```java
public class Iphone implements Phone{
  @Override
  public String brand(){
    return "this is a Apple phone";
  }
}

public class HuaWei implements Phone{
  @Override
  public String brand(){
    return "this is a HuaWei phone";
  }
}
```

#### 工厂类生产手机的工厂

```java
public class Factory{
  public Phone createPhone(String phoneName){
    if ("HuaWei".equals(phoneName)){
      return new HuaWei();
    } else if ("Apple".equals(phoneName)) {
      return new Iphone();
    }
  }
}
```

#### 测试类FactoryDemo

```java
public static void main(String[] args){
  Factory factory = new Factory();
  Phone huawei = factory.createPhone("HuaWei");
  Phone iphone = factory.createPhone("Apple");
  logger.info(huawei.brand());
  logger.info(iphone.brand());
}
```

