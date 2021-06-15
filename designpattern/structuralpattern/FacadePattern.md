# 外观模式

外观模式也叫作门面模式，通过一个门面向客户端提供一个访问系统的统一接口，客户端无须关心和知晓系统内部各子模块（系统）之间的复杂关系，其主要目的是降低访问拥有多个子系统的复杂系统的难度，简化客户端与其之间的接口。

外观模式将子系统中的功能抽象成一个统一的接口，客户端通过这个接口访问系统，使得系统使用起来更加容易。

如图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/外观模式设计图.png)

外观模式就是将多个子系统及其之间的复杂关系和调用流程封装到一个统一的接口或类中以对外提供服务。这种模式涉及3种角色：

* 子系统角色：实现了子系统的功能。
* 门面角色：外观模式的核心，熟悉各子系统的功能和调用关系并根据客户端的需求封装统一的方法来对外提供服务。
* 客户角色：通过调用Facade来完成业务功能。

如汽车的启动为例，类图如下：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/外观模式类图.png)

### 示例代码

1. #### 定义Dashboard类

```java
public class Dashboard {

    public void startup() {
        System.out.println("dashboard startup...");
    }

    public void shutdown() {
        System.out.println("dashboard shutdown...");
    }
}

```

2. #### 定义Engine类

```java
public class Engine {

    public void startup() {
        System.out.println("Engine startup...");
    }

    public void shutdown() {
        System.out.println("Engine shutdown...");
    }
}
```

3. #### 定义SelfCheck类

```java
public class SelfCheck {

    public void startupCheck() {
        System.out.println("startup check finished.");
    }

    public void shutdownCheck() {
        System.out.println("shutdown check finished.");
    }
}
```

4. #### 定义Starter类

```java
public class Starter {

    private Dashboard dashboard;
    private Engine engine;
    private SelfCheck selfCheck;

    public Starter() {
        this.dashboard = new Dashboard();
        this.engine = new Engine();
        this.selfCheck = new SelfCheck();
    }

    public void startup() {
        System.out.println("car engin startup.");
        engine.startup();
        dashboard.startup();
        selfCheck.shutdownCheck();
        System.out.println("car engin finish.");
    }

    public void shutdown() {
        System.out.println("car engine shutdown.");
        engine.shutdown();
        dashboard.shutdown();
        selfCheck.shutdownCheck();
        System.out.println("car engine shutdown finish.");
    }
}
```

5. #### 定义测试类

```java
public class MainTest {

    public static void main(String[] args) {
        Starter starter = new Starter();
        System.out.println("....startup..");
        starter.startup();
        System.out.println("....shutdown..");
        starter.shutdown();

    }
}
```

