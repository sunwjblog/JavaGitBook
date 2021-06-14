# 装饰器模式

装饰者模式指在无须改变原有类及类的继承关系的情况下，动态扩展一个类的功能。

它通过装饰者来包裹真实的对象，并动态地向对象添加或者撤销功能。

如图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/装饰者模式.png)

装饰者模式包括Source和Decorator两种角色，Source是被装饰者，Decorator是装饰者。通过装饰者模式通过装饰者可以为被装饰者Source动态添加一些功能。

#### 示例代码

1. #### 创建Sourceable接口

```java
public interface SourceAble {

    void createComputer();
}

```

2. #### 定义Sourceable接口的实现类Source

```java
public class Source implements SourceAble{
    @Override
    public void createComputer() {
        System.out.println("create Computer by Source");
    }
}
```

3. #### 定义装饰者类Decorator

```java
public class Decorator implements SourceAble{

    private Source source;

    public Decorator(Source source) {
        this.source = source;
    }

    @Override
    public void createComputer() {
        source.createComputer();
        System.out.println("购完电脑，然后装系统。");
    }
}
```

以上代码定义了装饰者类Decorator，装饰者类通过构造函数将Sourceable实例初始化到内部，并在其方法createComputer()中调用原方法后加上了装饰者逻辑，这里的装饰指在电脑创建完成后给电脑装上相应的系统。注意，之前的Sourceable没有给电脑安装系统的步骤，我们引入装饰者为Sourceable扩展了安装系统的功能。

4. #### 测试类

```java
public class MainTest {

    public static void main(String[] args) {
        Decorator decorator = new Decorator(new Source());
        decorator.createComputer();
    }
}
```

