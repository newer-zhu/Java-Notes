放在接口实现类或接口实现方法上

由于是基于Spring AOP，所以被注解的方法必须是public的

### 失效情况

1. A调用B，A没被注解而B注解了
2. 接口中异常没有被抛出而是被捕获
3. 多线程下事务不属于Spring托管
4. 类内部调用并不会被AOP捕获

一个被注解的方法里两次访问数据库，两次的connection必须是同一个，通过ThreadLocal可以获取到同一个connection