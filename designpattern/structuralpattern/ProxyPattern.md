# 代理模式

代理模式指为对象提供一种通过代理的方式来访问并控制该对象行为的方法。在客户端不适合或者不能够直接引用一个对象时，可以通过该对象的代理对象来实现对该对象的访问，可以将该代理对象理解为客户端和目标对象之间的中介者。

在代理模式下有两种角色，一种被代理者，一种时代理（Proxy），在被代理者需要做一项工作时，不用自己做，而是交给代理做。

我们生活中也有很多例子，比如企业在招人时，不用自己去市场上找，可以通过代理（猎头公司）去找，代理有候选人池，可根据企业的需求筛选出合适的候选人返回给企业。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/代理模式类图.png)

### 实现代码

1. #### 定义Company接口

```java
public interface Company {

    void findWorker(String title);
}
```

2. #### 定义HR实现类

```java
public class HR implements Company{
    @Override
    public void findWorker(String title) {
        System.out.println("i need a worker. title : " + title);
    }
}

```

3. #### 定义代理类Proxy

```java
public class Proxy implements Company{

    private HR hr;

    public Proxy() {
        this.hr = new HR();
    }

    @Override
    public void findWorker(String title) {
        hr.findWorker(title);
        String worker = this.getWorker(title);
        System.out.println("find a worker by proxy, worker name is : " + worker);
    }

    private String getWorker(String title) {

        Map<String,String> workerList = new HashMap<String,String>(){};
        workerList.put("Java","张三");
        workerList.put("Python","李四");
        workerList.put("Php","王五");

        return workerList.get(title);
    }
}
```

4. #### 测试类

```java
public class MainTest {

    public static void main(String[] args) {
        Company proxy = new Proxy();
        proxy.findWorker("Java");
    }
}
```

