# Spring循环依赖

### 循环依赖的分类

* 构造器的循环依赖
  * 构造器的循环依赖就是构造器中有属性循环依赖。如A类中的构造函数初始化B类实例，而B类中的构造函数又初始化A类的实例。
  * 构造器循环依赖，没有什么解决办法，因为JVM虚拟机在对类进行实例化的时候，需先实例化构造器的函数，而由于循环引用这个参数无法提前实例化。
* 属性的循环依赖
  * Spring解决的依赖是指属性的循环依赖。

### Spring解决的循环依赖

如：

```java
@Service
public class Teacher {
    @Autowired
    private Student student;

    public Teacher () {
        System.out.println("Teacher init1:" + student);

    }

    public void teach () {
        System.out.println("teach:");
        student.learn();
    }

}
```

```java
@Service
public class Student {
    @Autowired
    private Teacher teacher;

    public Student () {
        System.out.println("Student init:" + teacher);
    }

    public void learn () {
        System.out.println("Student learn");
    }
}
```

Spring解决的是这种属性的循环依赖，当加载bean的时候，Spring容器通过三级缓存的方式实现这种属性的循环依赖。**Spring通过将实例化后的对象提前暴露给Spring容器中的singletonFactories，解决了循环依赖的问题**。

#### 具体分析

1. 对于非懒加载的类，是在refresh方法中的 finishBeanFactoryInitialization(beanFactory) 方法完成的包扫描以及bean的初始化

```java
 protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
         // 其他代码
 
         // Instantiate all remaining (non-lazy-init) singletons.
         beanFactory.preInstantiateSingletons();
     }
```

可以看到调用了beanFactory的一个方法，此处的beanFactory就是指我们最常见的那个DefaultListableBeanFactory，继续追踪代码。

2. **DefaultListableBeanFactory的preInstantiateSingletons方法**

```java
public void preInstantiateSingletons() throws BeansException {

        List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

        // Trigger initialization of all non-lazy singleton beans...
        for (String beanName : beanNames) {
            RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
            if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) { // 判断为非抽象类、是单例、非懒加载 才给初始化
                if (isFactoryBean(beanName)) {
                    // 无关代码（针对FactoryBean的处理）
                }
                else {
                    // 重要！！！普通bean就是在这里初始化的
                    getBean(beanName);
                }
            }
        }

        // 其他无关代码
    }
```

在此方法中循环Spring容器中的所有bean，依次对其进行初始化，初始化的入口是这个getBean方法，继续追踪代码。

3. **AbstractBeanFactory的getBean跟doGetBean方法**

```java
 public Object getBean(String name) throws BeansException {
         return doGetBean(name, null, null, false);
     }
```

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
            @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

        final String beanName = transformedBeanName(name);
        Object bean;

         // 方法1）从三个map中获取单例类
        Object sharedInstance = getSingleton(beanName);
        // 省略无关代码
        }
        else {
            // 如果是多例的循环引用，则直接报错
            if (isPrototypeCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(beanName);
            }
            // 省略若干无关代码
            try {
                // Create bean instance.
                if (mbd.isSingleton()) {
                    // 方法2) 获取单例对象
                    sharedInstance = getSingleton(beanName, () -> {
                        try { //方法3) 创建ObjectFactory中getObject方法的返回值
                            return createBean(beanName, mbd, args);
                        }
                        catch (BeansException ex) {
                            // Explicitly remove instance from singleton cache: It might have been put there
                            // eagerly by the creation process, to allow for circular reference resolution.
                            // Also remove any beans that received a temporary reference to the bean.
                            destroySingleton(beanName);
                            throw ex;
                        }
                    });
                    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                }
         }
        // 省略若干无关代码
        return (T) bean;
    }
```

对于解决属性循环一用来说，是经过以上标注的3个方法进行处理的，以下分别对这三个方法进行分析。

*  **3.1 getSingleton(beanName)方法**： **注意该方法跟方法2）是重载方法，名字一样内部逻辑不一样。**

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        Object singletonObject = this.singletonObjects.get(beanName);// 步骤A
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            synchronized (this.singletonObjects) {
                singletonObject = this.earlySingletonObjects.get(beanName);// 步骤B
                if (singletonObject == null && allowEarlyReference) {
                    ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);// 步骤C
                    if (singletonFactory != null) {
                        singletonObject = singletonFactory.getObject();
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }
        return singletonObject;
    }
```

以上代码中出现的3个map，可以理解为3级缓存：

* singletonObjects里面存放的是初始化之后的单例对象。
* earlySingletonObjects中存放的是一个已完成实例化未完成初始化的早期单例对象。
* singletonFactories中存放的是ObjectFactory对象，此对象的getObject方法返回值即刚完成实例化还未开始初始化的单例对象。

它们的先后顺序是，**单例对象先存在于singletonFactories中，后存在于earlySingletonObjects中，最后初始化完成后放入singletonObjects中**。

以上述Teacher和Student两个循环引用的类为例，如果第一个走到这一步的是Teacher，则从此处这三个map中get到的值都是空，因为还未添加进去。这个方法主要是给循环依赖中后来过来的对象用。

* **3.2 getSingleton(String beanName, ObjectFactory<?> singletonFactory)方法**

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "Bean name must not be null");
        synchronized (this.singletonObjects) {
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                // 省略无关代码
                beforeSingletonCreation(beanName); // 步骤A
                boolean newSingleton = false;
                // 省略无关代码
                try {
                    singletonObject = singletonFactory.getObject();// 步骤B
                    newSingleton = true;
                }
                // 省略无关代码
                finally {
                    if (recordSuppressedExceptions) {
                        this.suppressedExceptions = null;
                    }
                    afterSingletonCreation(beanName);// 步骤C
                }
                if (newSingleton) {
                    addSingleton(beanName, singletonObject);// 步骤D
                }
            }
            return singletonObject;
        }
    }
```

**获取单例对象的主要逻辑就是此方法实现的**，主要分为上面四个步骤，继续分析

* 步骤A

```java
 protected void beforeSingletonCreation(String beanName) {
         // 判断，并首次将beanName即teacher放入singletonsCurrentlyInCreation中
         if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
             throw new BeanCurrentlyInCreationException(beanName);
         }
     }
```

* 步骤C

```java
 protected void afterSingletonCreation(String beanName) {
         // 得到单例对象后，再讲beanName从singletonsCurrentlyInCreation中移除
         if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
             throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
         }
     }
```

* 步骤D

```java
protected void addSingleton(String beanName, Object singletonObject) {
        synchronized (this.singletonObjects) {
            this.singletonObjects.put(beanName, singletonObject);//添加单例对象到map中
            this.singletonFactories.remove(beanName);//从早期暴露的工厂中移除，此map在解决循环依赖中发挥了关键的作用
            this.earlySingletonObjects.remove(beanName);//从早期暴露的对象map中移除
            this.registeredSingletons.add(beanName);//添加到已注册的单例名字集合中
        }
    }
```

* 步骤B

  此处调用了ObjectFactory的getObject方法，此方法是在哪里实现的呢？返回的又是什么？且往回翻，找到3中的方法3，对java8函数式编程有过了解的园友应该能看出来，方法3 【createBean(beanName, mbd, args)】的返回值就是getObject方法的返回值，即方法3返回的就是我们需要的单例对象，下面且追踪方法3而去。

* **3.3 AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[]) 方法**

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
            throws BeanCreationException {

        // 省略无关代码
        try {
            Object beanInstance = doCreateBean(beanName, mbdToUse, args);
            return beanInstance;
        }
        // 省略无关代码
    }
```

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
            throws BeanCreationException {

        BeanWrapper instanceWrapper = null;
        // 省略代码
        if (instanceWrapper == null) {
            // 实例化bean
            instanceWrapper = createBeanInstance(beanName, mbd, args);
        }
        boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                isSingletonCurrentlyInCreation(beanName));
        if (earlySingletonExposure) {
            // 重点！！！将实例化的对象添加到singletonFactories中
            addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
        }
        // 初始化bean
        Object exposedObject = bean;
        try {
            populateBean(beanName, mbd, instanceWrapper);//也很重要
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }
        // 省略无关代码
        return exposedObject;
}
```

上面注释中标出的重点是此方法的关键。在addSingletonFactory方法中，将第二个参数ObjectFactory存入了singletonFactories供其他对象依赖时调用。然后下面的**populateBean方法对刚实例化的bean进行属性注入**，如果遇到Spring中的对象属性，则再通过getBean方法获取该对象。至此，循环依赖在Spring中的处理过程已经追溯完毕。

### 小结

属性注入主要是在populateBean方法中进行的。对于循环依赖，以我们上文中的Teacher中注入了Student、Student中注入了Teacher为例来说明，假定Spring的加载顺序为先加载Teacher，再加载Student。

getBean方法触发Teacher的初始化后：

  a. 首先走到3中的方法1），此时map中都为空，获取不到实例；

  b. 然后走到方法2）中，步骤A、步骤C、步骤D为控制map中数据的方法，实现简单，可暂不关注。其中步骤B的getObject方法触发对方法3)的调用；

  c. 在方法3）中，先通过createBeanInstance实例化Teacher对象，又将该实例化的对象通过addSingletonFactory方法放入singletonFactories中，完成Teacher对象早期的暴露；

  d. 然后在方法3）中通过populateBean方法对Teacher对象进行属性的注入，发现它有一个Student属性，则触发getBean方法对Student进行初始化

  e. 重复a、b、c步骤，只是此时要初始化的是Student对象

  f. 走到d的时候，调用populateBean对Student对象进行属性注入，发现它有一个Teacher属性，则触发getBean方法对Teacher进行初始化；

  g. 对Teacher进行初始化，又来到a，但此时map已经不为空了，因为之前在c步骤中已经将Teacher实例放入了singletonFactories中，a中得到Teacher实例后返回；

  h.完成f中对Student的初始化，继而依次往上回溯完成Teacher的初始化；

完成Teacher的初始化后，Student的初始化就简单了，因为map中已经存了这个单例。

至此，Spring循环依赖的总结分析结束，一句话来概括一下：**Spring通过将实例化后的对象提前暴露给Spring容器中的singletonFactories，解决了循环依赖的问题**。

### 参考

[Spring中的循环依赖解决详解](https://www.cnblogs.com/zzq6032010/p/11406405.html)

