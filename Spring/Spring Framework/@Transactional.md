## Spring中关于事务的组件

事务的包有三个接口文件，TransactionStatus，TransactionDefinition，PlatformTransactionManager.

1. PlatformTransactionManager是事务平台管理器，用于管理事务，提供了3个事务操作的方法。
   	commit，rollback，getTransaction
   它并不知道底层实现，由实现类具体去实现。
2. TransactionDefinition定义了事务规则，提供了提取事务信息的方法
3. TransactionStatus描述了某一时间的事务状态

## 注解

**由于是基于Spring AOP，所以被注解的方法必须是public的**

### 失效情况

1. A调用B，A没被注解而B注解了
2. 接口中异常没有被抛出而是被捕获
3. 多线程下事务不属于Spring托管
4. 类内部调用并不会被AOP捕获。就调该类自己的方法，而没有经过 Spring 的代理类
5. 类没有被Spring管理
6. 数据库不支持事务
7. 异常类型错误
8. 数据源没有配置事务管理器

一个被注解的方法里两次访问数据库，两次的connection必须是同一个，通过ThreadLocal可以获取到同一个connection

