# 适配器模式

我们在开发中遇到各个系统之间的对接问题，然而每个系统的数据模型或多或少存在差别，因此可能存在修改现有对象模型的情况，这就影响到了系统的稳定。

若想在不修改原有代码结构（类的结构）的情况下完成友好对接，就需要用到适配器模式。

适配器模式通过定义一个适配器类作为两个不兼容的接口之间的桥梁，将一个类的接口转换成用户期望的另一个接口，使得两个或多个原本不兼容的接口可以基于适配器类一起工作。

如图

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/适配器模式表示图.png)

在适配器模式的是实现中有三个角色：Source、Targetable、Adapter。

* Source是适配的类
* Targetable是目标接口
* Adapter是适配器

我们在具体应用中通过Adapter将Source的功能扩展到Targetable，以实现接口的兼容。适配器的实现主要分为三类：类适配模式、对象适配器模式、接口适配器模式。

### 类适配器模式

如图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/类适配器模式类图.png)

#### 示例代码

1. #### 创建Source类

```java
public class Source {

    public void editFile() {
        System.out.println("a text file editing");
    }
}
```

2. #### 创建目标接口Targetable

```java
public interface Targetable {

    void editTextFile();

    void editWordFile();
}
```

3. #### 创建适配器类Adapter

```java
// 继承带适配的类并且实现目标接口类 这样适配器就具有两者的共同行为
public class Adapter extends Source implements Targetable{

    @Override
    public void editTextFile() {
        super.editFile();
    }

    @Override
    public void editWordFile() {
        System.out.println(" a word file editing");
    }
}
```

4. #### 创建测试类MainTest

```java
public class MainTest {

    public static void main(String[] args) {
        Targetable targetable = new Adapter();
        targetable.editTextFile();
        targetable.editWordFile();
    }
}
```

### 对象适配器模式

如图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/对象适配器模式.png)

#### 示例代码

1. #### 创建Source类

```java
public class Source {

    public void editFile() {
        System.out.println("a text file editing");
    }
}
```

2. #### 创建目标接口Targetable

```java
public interface Targetable {

    void editTextFile();

    void editWordFile();
}
```

3. #### 创建对象适配器类ObjectAdapter

```java
public class ObjectAdapter implements Targetable {

    private Source source;

    public ObjectAdapter(Source source) {
        super();
        this.source = source;
    }

    @Override
    public void editTextFile() {
        this.source.editFile();
    }

    @Override
    public void editWordFile() {
        System.out.println(" a word file editing");
    }
}
```

4. #### 创建测试类MainTest

```java
public class MainTest {

    public static void main(String[] args) {
        Source source = new Source();
        Targetable targetable = new ObjectAdapter(source);
        targetable.editTextFile();
        targetable.editWordFile();
    }
}
```

### 接口适配器模式

如图

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/接口适配器模式.png)

#### 示例代码

1. #### 定义公共接口Sourceable

```java
public interface SourceAble {

    void editTextFile();

    void editWordFile();
}

```

2. #### 定义抽象类AbstractAdapter并实现公共接口方法

```java
public abstract class AbstractAdapter implements SourceAble {

    // 定义了SourceAble的抽象实现类AbstractAdapter，该类对SourceAble进行重写，
    // 但是不做具体实现。

    @Override
    public void editTextFile() {

    }

    @Override
    public void editWordFile() {

    }
}
```

3. #### 定义SourceSub1类按照需求实现editTextFile方法

```java
public class SourceSub1 extends AbstractAdapter{
    @Override
    public void editTextFile() {
        System.out.println("a text file editing");
    }
}
```

4. #### 定义SourceSub2类按照需求实现editWordFile方法

```java
public class SourceSub2 extends AbstractAdapter{

    @Override
    public void editWordFile() {
        System.out.println("a word file editing");
    }
}
```

5. #### 测试类

```java
public class MainTest {

    public static void main(String[] args) {

        SourceAble source1 = new SourceSub1();
        SourceAble source2 = new SourceSub2();
        source1.editTextFile();
        source2.editWordFile();
    }
}
```

