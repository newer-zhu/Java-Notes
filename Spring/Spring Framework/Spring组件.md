**BeanDefination**(@Component 、 @Bean、 <bean/>会被解析为此对象)

- beanClass

- scope

- isLazy

- dependson

- primary

- initMethodName

**BeanFactory**（BeanFactory利用BeanDefination生成Bean,延时加载）

核心子接口和实现类

- DefaultListableBeanFactory
- 支持单例Bean，Bean别名，父子BeanFactory。Bean类型转化，Bean后置处理，FactoryBean，自动装配等
- ConfigurableBeanFactory
- AutowireCapableBeanFactory
- AbsractBeanFactory
- DefaultListableBeanFactory

==为什么要定义多层次的子接口？==

区分再spring内部操作对象的传递和转化，对对象的数据访问所作的限制。

**FactoryBean**

Spring提供的创建Bean的方式，通过此接口的getObject方法返回最终的Bean对象，还有isSingleton， getObjectType方法

广泛运用在Spring与第三方框架组件的整合过程中，FactoryBean本身是一个Bean，可以通过实现 FactoryBean 接口定制实例化 Bean 的逻辑，通过代理一个Bean对象，对方法前后做一些操作。

而BeanFactory是一个Spring容器，可以生产和管理Bean

**ApplicationContext**

是Beanfactory的子接口，但比Beanfactory更强大，可以获取创建Bean，支持国际化，事件广播，获取资源等功能

主要用来规范容器中bean对象的非延迟加载，即创建容器对象的时候就对bean初始化。

继承的接口

![image-20210725023509089](E:\学习笔记\typora\img\image-20210725023509089.png)

- 拥有获取环境变量的功能
- 获取所有beanNames，判断某个beanName是否有BeanDefination对象和统计BeanDefination个数等功能
- 获取父BeanFactory的功能
- 国际化功能
- 事件发布功能
- 加载并获取资源的功能

ApplicationContext的三个常用实现类

1. ClassPathXmlApplicationContext可以加载类路径下的配置文件
2. FileSystemXMLApplicationContext加载磁盘下的配置文件
3. AnnotationConfigXMLApplicationContext读取注解创建容器

**BeanPostProcessor**（Bean后置处理器）

Spring提供的扩展机制，允许对Bean定制加工，是一个接口

方法有初始化前和初始化后两个方法

子接口有InstantiationAwareBeanPostProcessor，包含了实例化前，实例化后，属性注入后三个方法

**Bean生命周期**

![image-20210725023627979](E:\学习笔记\typora\img\image-20210725023627979.png)

AOP  在Spring实例化出来一个Target对象后进行属性填充 ，初始化后会判断Target对象有没有对应的切面，有则进行AOP，通过动态代理生成最终对象

**@Resource & @Autowired**

找到@Resource注解后，判断其name属性是否为空，若为空，看Spring容器中的bean中的id与@Resource要注解的那个变量属性名是否相同，如相同，匹配成功；如不同，看spring容器中bean的id对应的类型是否与@Resource要注解的那个变量属性对应的类型是否相等，若相等，匹配成功，若不相等，匹配失败

@Autowired 默认是按照类去匹配，配合 @Qualifier 指定按照名称去装配 bean

@Autowired可以指定实例化bean所用的构造方法，当有多个构造方法时。

也可以标在setter和getter方法上实现属性注入

**@Value**

@Value("xxx") 注入字符串

@Value("${xxx}) 注入配置文件的属性

@Value("#{xxx}) 注入容器中的bean

