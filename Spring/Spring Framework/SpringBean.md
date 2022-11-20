### 创建Bean的方式

1. 配置文件中使用bean标签，只有id和class属性，此时类中必须要有默认构造函数

2. ==声明式== @Bean，@ComponentScan扫描路径或者@Component注解生成Bean对象

3. ==编程式== 利用BeanDefinitionBuilder

   ![image-20210727012442029](E:\学习笔记\typora\img\image-20210727012442029.png)

4. 自定义类实现FactoryBean接口并加上@Bean，getObject方法返回自定义对象Bean。可以根据这个接口在Spring中放入代理后的bean，通常在三方框架整合中运动。

5. BeanFactory和Application都有registerBean 方法注册一个Bean，底层还是BeanDefinition构造。参数为Supplier接口，重写里面的get方法

### Bean的生命周期

![Bean的生命周期](https://uploadfiles.nowcoder.com/images/20220224/4107856_1645694380479/7EF8F66C3DFA7434E4CA11B47CF8F1F7)

#### 测试代码

```java
public class Person implements BeanFactoryAware, BeanNameAware, InitializingBean, DisposableBean {
    private int phone;
    private BeanFactory beanFactory;
    private String beanName;

    public Person() {
        System.out.println("【构造器】调用Person的构造器实例化");
    }

    public int getPhone() {
        return phone;
    }

    public void setPhone(int phone) {
        System.out.println("【注入属性】phone");
        this.phone = phone;
    }

    // 这是BeanFactoryAware接口方法
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("【BeanFactoryAware接口】调用setBeanFactory方法");
        this.beanFactory = beanFactory;
    }

    // 这是BeanNameAware接口方法
    public void setBeanName(String s) {
        System.out.println("【BeanNameAware接口】调用setBeanName方法");
        this.beanName = s;
    }

    // 这是DiposibleBean接口方法
    public void destroy() throws Exception {
        System.out.println("【DiposibleBean接口】调用destroy方法");
    }

    // 这是InitializingBean接口方法
    public void afterPropertiesSet() throws Exception {
        System.out.println("【InitializingBean接口】调用afterPropertiesSet方法");
    }

    // 通过<bean>的init-method属性指定的初始化方法
    public void myInit() {
        System.out.println("【init-method】调用<bean>的init-method属性指定的初始化方法");
    }

    // 通过<bean>的destroy-method属性指定的初始化方法
    public void myDestory() {
        System.out.println("【destroy-method】调用<bean>的destroy-method属性指定的初始化方法");
    }
}
```

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor{
    public MyBeanFactoryPostProcessor() {
        super();
        System.out.println("这是BeanFactoryPostProcessor实现类构造器！！");
    }

    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out
                .println("BeanFactoryPostProcessor调用postProcessBeanFactory方法");
        BeanDefinition bd = configurableListableBeanFactory.getBeanDefinition("person");
        bd.getPropertyValues().addPropertyValue("phone", "110");
    }
}
```

```java
public class MyBeanPostProcessor implements BeanPostProcessor{
    public MyBeanPostProcessor(){
        System.out.println("这是BeanPostProcessor实现类构造器！！");
    }
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改");
        return o;
    }

    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改");
        return o;
    }
}
```



```java
public class MyInstantiationAwareBeanPostProcessor extends
        InstantiationAwareBeanPostProcessorAdapter {

    public MyInstantiationAwareBeanPostProcessor() {
        super();
        System.out
                .println("这是InstantiationAwareBeanPostProcessorAdapter实现类构造器！！");
    }

    // 接口方法、实例化Bean之前调用
    @Override
    public Object postProcessBeforeInstantiation(Class beanClass,
                                                 String beanName) throws BeansException {
        System.out
                .println("InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法");
        return null;
    }

    // 接口方法、实例化Bean之后调用
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
        System.out
                .println("InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法");
        return bean;
    }

    // 接口方法、设置某个属性时调用
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs,
                                                    PropertyDescriptor[] pds, Object bean, String beanName)
            throws BeansException {
        System.out
                .println("InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法");
        return pvs;
    }
}
```



1. BeanFactoryPostProcessor实例化

2. BeanFactoryPostProcessor调用postProcessBeanFactory方法

   

3. BeanPostProcessor实例化

   

4. InstantiationAwareBeanPostProcessorAdapter实例化

5. InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation

6. InstantiationAwareBeanPostProcessor调用postProcessPropertyValues

   

7. BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改

8. BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改

   

9. InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法

10. InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法

    ----------------------------------下面是回答模板---------------------------------

11. 【Bean实例化】利用反射调用Person的构造器实例化（推断构造方法）*  *createBeanInstance方法创建*

12. InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法

13. 【注入属性】*  *利用反射实例化后只是分配内存空间，并没有对属性进行设置。polulateBean方法注入*

    

14. 【BeanNameAware接口】调用setBeanName方法     * *invokeAwareMethds设置容器对象*

15. 【BeanFactoryAware接口】调用setBeanFactory方法    * Aware接口是为了自定义对象能够使用Spring的内部对象，如BeanFactory

16. BeanPostProcessor接口postProcessBeforeInitialization方法对属性进行更改 *

    

17. 【InitializingBean接口】调用afterPropertiesSet方法  *

18. 【init-method】调用<bean>的init-method属性指定的初始化方法   *

19. BeanPostProcessor接口方法postProcessAfterInitialization调用   *

    ----------------此时bean已经可以用了----------

20. InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法

    

21. 容器初始化成功

22. 现在开始关闭容器！

23. 【DiposibleBean接口】调用destroy方法    *

24. 【destroy-method】调用<bean>的destroy-method属性指定的初始化方法    *

#### 各组件作用

**BeanFactoryPostProcessor**

```java
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory var1) throws BeansException;
}
```

在bean factory标准初始化之后可以进行修改。将加载所有bean定义，但是还没有实例化bean。这个方法允许重新覆盖或者添加属性甚至快速的初始化bean。

**BeanPostProcessor**

后置处理器，一个接口。方法返回true和false可以控制是否继续走spring的生命周期逻辑

- postProcessBeforeInitialization(Object bean, String beanName)：在一些bean实例化回调
- postProcessAfterInitialization(Object bean, String beanName)

**InstantiationAwareBeanPostProcessorAdapter**

对已经实例化的bean进行一些处理

**BeanNameAware**

- setBeanName(String name)：在创建这个bean的bean factory里设置名字。在填充正常bean属性之后调用但是在初始化回调之前例如InitializingBean的afterPropertiesSet方法或者一个定制的init-method.

**BeanFactoryAware**

- setBeanFactory(BeanFactory beanFactory) :为bean实例提供所属工厂的回调。在普通的bean属性值填充之后但是在初始化回调之前（例如InitializingBean的afterPropertiesSet方法或者一个定制的init-method方法）被调用

**InitializingBean**

- afterPropertiesSet()：在设置完所有提供的bean属性（并满足BeanFactoryAware和ApplicationContextAware）之后由beanFactory调用。这个方法允许bean实例只有在所有的bean属性都被设置并且在错误配置的情况下抛出异常的情况下才能执行初始化。

**DisposableBean**

接口已经被实现由那些想在销毁释放资源的bean。如果BeanFactory处理缓存的单例对象，那么它应该调用destroy方法。 
应用程序上下文在关闭的时候应该处理它的所有单例。

### 为什么Bean是单例的？

1. **减少了新生成bean的消耗** 新生成实例消耗包括两方面，第一，Spring会通过反射或者 cglib来生成 bean实例，这都是耗性能的操作，其次给对象分配内存也会涉及复杂算法

2. **减少jvm垃圾回收** 由于不会给每个请求都新生成bean实例，所以自然回收的对象少了

3. **可以快速获取到 bean** 因为单例的获取bean操作除了第一次生成之外其余的都是从缓存里获取的，所以很快


### Bean是如何解决线程安全？

如果单例Bean,是一个无状态Bean，也就是线程中的操作不会对Bean的成员执行查询以外的操作，那么这个单例Bean是线程安全的。比如Spring mvc 的 Controller、Service、Dao等，这些Bean大多是无状态的，只关注于方法本身。

**有状态对象(Stateful Bean)** ：就是有实例变量的对象，可以保存数据，是非线程安全的。每个用户有自己特有的一个实例，在用户的生存期内，bean保持了用户的信息，即“有状态”；一旦用户灭亡（调用结束或实例结束），bean的生命期也告结束。即每个用户最初都会得到一个初始的bean。

**无状态对象(Stateless Bean)**：就是没有实例变量的对象，不能保存数据，是不变类，是线程安全的。bean一旦实例化就被加进会话池中，各个用户都可以共用。即使用户已经消亡，bean 的生命期也不一定结束，它可能依然存在于会话池中，供其他用户调用。由于没有特定的用户，那么也就不能保持某一用户的状态，所以叫无状态bean。但无状态会话bean 并非没有状态，如果它有自己的属性（变量），那么这些变量就会受到所有调用它的用户的影响，这是在实际应用中必须注意的。

**对于有状态的bean，Spring官方提供的bean，一般提供了通过ThreadLocal去解决线程安全的方法**

### IOC周期

1. 低级容器 加载配置文件（从 XML，数据库，Applet），并解析成 `BeanDefinition` 到低级容器中。
2. 加载成功后，高级容器启动高级功能，例如接口回调，监听器，自动实例化单例，发布事件等等功能。
