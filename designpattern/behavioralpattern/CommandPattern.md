# 命令模式

命令模式指将请求封装为命令基于事件驱动异步地执行，以实现命令的发送者和命令的执行者之间的解藕，提高命令发送、执行的效率和灵活度。

命令模式将命令调用者与命令执行者解藕，有效降低系统的耦合度。同时，由于命令调用者和命令执行者进行解耦，所以增加和删除（回滚）命令变得非常方便。

命令模式包含如下主要角色：

* 抽象命令类（Command）：执行命令的接口，定义执行命令的抽象方法execute()。
* 具体命令类（Concrete Command）：抽象命令类的实现类，持有接收者对象，并在接收到命令后调用命令执行者的方法action()实现命令的调用和执行。
* 命令执行者（Receiver）：命令的具体执行者，定义了命令执行的具体方法action()。
* 命令调用者（Invoker）：接收客户端的命令并异步执行。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/命令模式类图.png)

### 示例代码

* 定义Command接口

```
public interface Command {
	void exe(String command);
}
```

* 定义实现Command接口的实现类ConcreteCommand

```
public class ConcreteCommand implements Command {
	
	private Receiver receiver;
	public ConcreteCommand(){
		this.receiver = receiver;
	}
	
	@Override
	public void exe(String command) {
	
		receiver.action(command);
	}
}
```

* 定义命令调用者类Invoker

```
public class Invoker {

	private Command command;
	public Invoker(Command command) {
	
		this.command = command;
	}
	
	public void action(String commandMessage) {
	
		System.out.println("Command sending....");
		command.exe(commandMessage);
	}
}
```

* 定义命令的接收和执行者类Receiver

```
public class Receiver {

	public void action(String command) {
	
	 	// 接收并执行命令
	 	System.out.println("command received, now execute command");
	}
}
```

* 使用命令模式

```
public class CommandTest {

    public static void main(String[] args) {
        // 定义命令的接收和执行者
        Receiver receiver = new Receiver();
        // 定义命令实现者
        Command command = new ConcreteCommand(receiver);
        // 定义命令的调用者
        Invoker invoker = new Invoker(command);
        invoker.action("command1");
    }
}
```

