# 责任链模式

责任链模式也叫作职责链模式，用于避免请求发送者与多个请求处理者耦合在一起，让所有请求的处理者持有下一个对象的引用，从而将请求串联成一条链，在有请求发生时，可将请求沿着这条链传递，直到遇到该对象的处理器。

在责任链模式下，用户只需将请求发送到责任链上即可，无须关心请求的处理细节和传递过程，所以责任链模式优雅地将请求和处理进行了解藕。

如在Web请求中，提供的REST服务，服务端要针对客户端的请求实现用户鉴权、业务调用、结果反馈流程，就可以使用责任链模式。

责任链模式包含以下三种角色：

* Handler接口：用于规定在责任链上具体要执行的方法。
* AbstractHandler抽象类：持有Handler实例并通过setHandler()和getHandler()将各个具体的业务Handler串联成一个责任链，客户端上的请求在责任链上执行。
* 业务Handler：用户根据具体的业务需求实现的业务逻辑。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/责任链模式类图.png)

### 示例代码

* 定义Handler接口

```java
public interface Handler {

	void operator();
}
```

* 定义AbstractHandler类

```java
public abstract class AbstractHandler {

	private Handler handler;
	
	 ...setter getter方法
}
```

* 定义AuthHandler类

```java
public class AuthHandler extends AbstractHandler implements Handler {

	private String name;
	public AuthHandler(String name) {
		this.name = name;
	}
	
	@Override
	public void operator() {
		System.out.println("user auth....");
		if (getHandler()!=null) {
			// 执行责任链的下一个流程
			getHandler().operator();
		}
	}
}
```

* 定义业务处理类BusinessHandler

```java
public class BusinessHandler extends AbstractHandler implements Handler {

	private String name;
	public BusinessHandler(String name) {
		this.name = name;
	}
	
	@Override
	public void operator() {
		System.out.println("business info handler....");
		if (getHandler()!=null) {
			// 执行责任链的下一个流程
			getHandler().operator();
		}
	}
}
```

* 定义请求反馈类ResponseHandler

```java
public class ResponseHandler extends AbstractHandler implements Handler {

	private String name;
	public ResponseHandler(String name) {
		this.name = name;
	}
	
	@Override
	public void operator() {
		System.out.println("message response....");
		if (getHandler()!=null) {
			// 执行责任链的下一个流程
			getHandler().operator();
		}
	}
}
```

* 使用责任链模式

```java
public class ChainTest {

    public static void main(String[] args) {
        AuthHandler authHandler = new AuthHandler("auth");
        BusinessHandler businessHandler = new BusinessHandler("business");
        ResponseHandler responseHandler = new ResponseHandler("response");
        authHandler.setHandler(businessHandler);
        businessHandler.setHandler(responseHandler);
        authHandler.operator();
    }
}
```

