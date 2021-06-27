# 观察者模式

观察者模式指在被观察者的状态发生变化时，系统基于事件驱动理论将其状态通知到订阅其状态的观察者对象中，以完成状态的修改和事件传播。这种模式有时又叫做发布-订阅模式或者模型-视图模式。

观察者模式是一种对象行为型模式，观察者和被观察者之间的关系属于抽象耦合关系，主要优点是在观察者与被观察者之间建立了一套事件触发机制，以降低二者之间的耦合度。

观察者模式的主要角色如下：

* 抽象主题：持有订阅了该主题的观察者对象的集合，同时提供了增加、删除观察者对象的方法和主题状态发生变化后的通知方法。
* 具体主题：实现了抽象主题的通知方法，在主题的内部状态发生变化时，调用该方法通知订阅了主题状态的观察对象。
* 抽象观察者：观察者的抽象类或接口，定义了主题状态发生变化时需要调用的方法。
* 具体观察者：抽象观察者的实现类，在收到主题状态变化的信息后执行具体的触发机制。

如UML类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/观察者模式类图.png)

### 示例代码

* 定义抽象主题Subject

```java
public abstract class Subject {

	protected List<Observer> observers = new ArrayList<Observer>();
	
	// 增加观察者
	
	public void add(Observer observer) {
	
		observers.add(observer);
	
	}
	
	// 删除观察者
	
	public void remove(Observer observer) {
	
		observers.remove(observer);
		
	}
	
	public abstract void notifyObserver(String message); // 通知观察者的抽象方法
	
}
```

* 定义具体的主题ConcreteSubject：

```java
public class ConcreteSubject extends Subject {

	public void notifyObserver(String message) {
	
		for(Object obs : observers) {
		
			sout("notify observer "+message+" change...");
			((Observer)obs).dataChange(message);
		
		}
	
	}

}
```

* 定义抽象观察者Observer：

```java
public interface Observer {

	void dataChange(String message);// 接收数据

}
```

* 定义具体的观察者ConcreteObserver：

```java
public class ConcreteObserver implements Observer {

	public void dataChange(String message) {
	
		sout("recive meaage:" + message);
	
	}

}
```

* 使用观察者模式

```java
public static void main(String[] args) {

	Subject subject = new ConcreteSubject();
	Observer obs = new ConcreteObserver();
	subject.add(obs);
	subject.notifyObserver("data1");

}
```

