# 策略模式

策略模式为同一个行为定义了不同的策略，并为每种策略都实现了不同的方法。在用户使用的时候，系统根据不同的策略自动切换不同的方法来实现策略的改变。同一个策略下的不同方法是对同一功能的不同实现，因此在使用时可以相互替换而不影响用户的使用。

策略模式的实现时在接口中定义不同的策略，在实现类中完成了对不同策略下具体行为的实现，并将用户的策略状态存储在上下文中来完成策略的存储和状态的改变。

可以用于去除使用多重if...else条件转移语句。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/策略模式类图.png)

1. #### 定义TravelStrategy接口

```java
public interface TravelStrategy {

    void travelMode();
}

```

2. #### 定义TravelStrategy的两种实现方式TravelByAirStrategy和TravelByCarStrategy

```java
public class TravelByAirStrategy implements TravelStrategy{
    @Override
    public void travelMode() {
        System.out.println("travel by air");
    }
}
```

```java
public class TravelByCarStrategy implements TravelStrategy{
    @Override
    public void travelMode() {
        System.out.println("tavel by car");
    }
}
```

3. #### 定义Context实现策略模式

```java
public class Context {

    private TravelStrategy travelStrategy;

    public TravelStrategy getTravelStrategy() {
        return travelStrategy;
    }

    public void setTravelStrategy(TravelStrategy travelStrategy) {
        this.travelStrategy = travelStrategy;
    }

    public void travelMode() {
        this.travelStrategy.travelMode();
    }
}
```

4. #### 使用策略模式

```java
public class StrategyTest {

    public static void main(String[] args) {

        Context context = new Context();
        context.setTravelStrategy(new TravelByAirStrategy());
        context.travelMode();

        context.setTravelStrategy(new TravelByCarStrategy());
        context.travelMode();
    }
}
```

