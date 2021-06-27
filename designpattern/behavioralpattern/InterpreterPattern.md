# 解释器模式

解释器模式给定一种语言，并定义该语言的语法表示，然后设计一个解释器来解释语言中的语法，这种模式常被用于SQL解析、符号处理引擎等。

解释器模式包含如下角色：

* 抽象表达式：定义解释器的接口，约定解释器所包含的操作，比如interpret()方法。
* 终结符表达式：抽象表达式的子类，用来定义语法中和终结符相关的操作，语法中的每一个终结符都应有一个与之对应的终结表达式。
* 非终结符表达式：抽象表达式的子类，用来定义语法中和非终结符有关的操作，语法中的每条规则都有一个非终结符表达式与之对应。
* 环境：定义各个解释器需要的共享数据或者公共的功能。

如UML图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/解释器模式类图.png)

### 示例代码

* 定义Expression接口：

```java
public interface Expression {

	void interpret(Context ctx); // 解释方法
}
```

* 定义NonterminalExpression类

```java
public class NonterminalExpression implements Expression {

	private Expression left;
	private Expression right;
	
	public NonterminalExpression(Expression left, Expression right) {
	
		this.left = left;
		this.right = right;
	}
	public void interpret(Context ctx) {
		
		// 递归调用每一个组成部分的interpret()
		// 在递归调用时制定组成部分的连接方式，即非终结符的功能
	
	}

}
```

* 定义TerminalExpression类

```java
public class TerminalExpression implements Expression {

	@Override
	public void interpret(Context ctx) {
	
		// 终结符表达式的解释操作
	}

}
```

* 定义Context类

```java
public class Context {

	private HashMap map = new HashMap();
	
	public void assign(String key, String value) {
	
		// 在环境类中设值
	
	}
	
	public String get(String key) {
	
		// 获取存储在环境类中的值
		return "";
	}

}
```

