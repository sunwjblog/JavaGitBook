# Java反射

### 什么是Java反射？

> Java的反射(reflection)机制是指在程序的运行状态中，可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。 -- 来自百度百科

反射是动态语言的关键。

自己的理解，Java反射就是在我们运行成语过程中，可以逆向的构造一个我们所需的对象，然后可以访问该对象的类，方法和属性等。

Java反射在许多优秀的框架中得以应用，比如Spring的AOP动态代理机制，Mybatis的动态代理机制，本质都是通过Java反射实现的。

### Java反射的应用场景有哪些？

Java反射在Spring/Spring Boot、Mybatis等框架中大量使用了反射机制。在框架中也大量了使用了动态代理，而动态代理的实现也依赖于反射机制。除此之外注解的实现也用到了Java反射机制。

举例JDK实现的动态代理的实现代码，其中使用了反射类的Method来调用制定的方法。

```java
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * 代理类中的真实对象
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }


    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after method " + method.getName());
        return result;
    }
}
```

在我们使用Spring框架的时候，一个@Service注解就声明了一个类为Spring Bean，一个@Value注解就可以读到配置文件中的值了，这些是为什么呢？是因为我们基于反射分析类，燃火获取到类/属性/方法/方法的参数上的注解。获取注解之后，就可以做下一步处理。

### Java反射的使用

#### Java反射获取Class对象有四种方式

* ##### 知道具体类的情况下可以使用

  ```
  Class alunbarClass = TargetObject.class;
  ```

* ##### 通过Class.forName()传入类的路径获取

  ```
  Class alunbarClass1 = Class.forName("cn.sunwj.TargetObject");
  ```

* ##### 通过对象实例 instance.getClass()获取

  ```
  TargetObject o = new TargetObject();
  Class alunbarClass2 = o.getClass();
  ```

* ##### 通过类加载器xxxClassLoader.loadClass()传入类路径获取

  ```
  class clazz = ClassLoader.LoadClass("cn.sunwj.TargetObject");
  ```

  通过类加载器获取 Class 对象不会进行初始化，意味着不进行包括初始化等一些列步骤，静态块和静态对象不会得到执行

* ##### 反射相关的一些API使用

  1. 创建一个我们要使用的反射操作的类，TargetObject。

     ```java
     public class TargetObject {
         private String value;
     
         public TargetObject() {
             value = "Sunwj";
         }
     
         public void publicMethod(String s) {
             System.out.println("I love " + s);
         }
     
         private void privateMethod() {
             System.out.println("value is " + value);
         }
     }
     ```

  2. 通过使用Java反射API操作这个类

     ```java
     public class Main {
         public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException, NoSuchFieldException {
             /**
              * 获取TargetObject类的Class对象并且创建TargetObject类实例
              */
             Class<?> tagetClass = Class.forName("com.Sunwj.TargetObject");
             TargetObject targetObject = (TargetObject) tagetClass.newInstance();
             /**
              * 获取所有类中所有定义的方法
              */
             Method[] methods = tagetClass.getDeclaredMethods();
             for (Method method : methods) {
                 System.out.println(method.getName());
             }
             /**
              * 获取指定方法并调用
              */
             Method publicMethod = tagetClass.getDeclaredMethod("publicMethod",
                     String.class);
     
             publicMethod.invoke(targetObject, "Sunwj");
             /**
              * 获取指定参数并对参数进行修改
              */
             Field field = tagetClass.getDeclaredField("value");
             //为了对类中的参数进行修改我们取消安全检查
             field.setAccessible(true);
             field.set(targetObject, "Suwnj");
             /**
              * 调用 private 方法
              */
             Method privateMethod = tagetClass.getDeclaredMethod("privateMethod");
             //为了调用private方法我们取消安全检查
             privateMethod.setAccessible(true);
             privateMethod.invoke(targetObject);
         }
     ```

     输出的内容

     ```
     publicMethod
     privateMethod
     I love Sunwj
     value is Sunwj
     ```

### Java反射的优缺点

* #### 优点

  * 可以让写的代码更加灵活，代码简洁，提高代码的复用率，外部调用方便。
  * 对于任意一个类，都能够知道这个类的所有属性和方法；
  * 对于任意一个对象，都能够调用它的任意一个方法

* #### 缺点

  * 让我们在运行时有了分析操作类的能力，这同样也增加了安全问题。比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）。
  * 反射的性能也要稍差点，不过，对于框架来说实际是影响不大的。

  

