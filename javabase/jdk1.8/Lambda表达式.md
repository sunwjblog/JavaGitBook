# Lambda表达式

[toc]

## Lambda管中窥豹

### Lambda表达式

#### Lambda表达式初识

Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：它可以没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。

* 匿名：说匿名，是因为它不像普通的方法那样有一个明确的名称：写得少而想的多！
* 函数：说它是函数，是因为Lambda函数不像方法那样属于某个特定的类。但和方法一样，Lambda有参数列表、函数主体、返回类型，还可能有可以抛出的异常列表。
* 传递：Lambda表达式可以作为参数传递给方法或存储在变量中。
* 简洁：无需像匿名类那样写的很多模版代码。

如：

内部类：

```java
Comparator<Apple> byWeight = new Comparator<Apple>() {

	public int compare(Apple a1, Apple a2) {
	
		return a1.getWeight().compareTo(a2.getWeight());
		
	}
	
}
```

使用Lambda表达式：

```java
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

#### Lambda表达式定义

如图：

![](../../image/Lambda语法.png)

* 参数列表：这里它采用Comparator中的compare方法的参数，两个Apple。
* 箭头：箭头 --> 把参数列表与Lambda主体分隔开。
* Lambda主体：比较两个Apple的重量。表达式就是Lambda的返回值了。

**有效的Lambda表达式：**

```java
// Lambda表达式具有一个String类型的参数并返回一个int。Lambda没有return语句，因为意境隐含了return
(String s) -> s.length() 

// Lambda表达式有一个Apple类型的参数并返回一个boolean（苹果的重量是否超过150克）
(Apple a) -> a.getWeight() > 150

//Lambda表达式具有两个int类型的参数没有返回值（void返回）。注意Lambda表达式可以包含多行语句，这里是两行
(int x, int y) -> {
	
	sout("Result:");
	sout(x+y);

}

// Lambda表达式没有参数，返回一个int
() -> 42

// Lambda表达式具有两个Apple类型的参数，返回一个int：比较两个Apple的重量 
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
```

**Lambda的基本语法：**

```
(parameters) -> expression
```

或（注意语句的花括号）

```
(parameters) -> { statement; }
```

**Lambda示例：**

| 使用案例              | Lambda示例                                                   |
| --------------------- | ------------------------------------------------------------ |
| 布尔表达式            | (List<String> list) -> list.isEmpty()                        |
| 创建对象              | () -> new Apple(10)                                          |
| 消费一个对象          | (Apple a) -> { sout(a.getWeight());}                         |
| 从一个对象中选择/抽取 | (String s) -> s.length()                                     |
| 组合两个值            | (int a, int b) -> a * b                                      |
| 比较两个对象          | (Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight()) |

## 在哪里以及如何使用Lambda



## 环绕执行模式



## 函数式接口，类型推断



## 方法引用



## Lambda复合
