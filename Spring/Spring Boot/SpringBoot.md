### @SpringBootApplication的合成注解

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

#### 其中 @EnableAutoConfiguration 有两个合成注解，可以完成自动配置

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})  
```

##### @AutoConfigurationPackage里面的注解及作用

```java
@Import({Registrar.class}) //给容器中导入一个组件
//利用Register给容器中导入一系列组件
//将指定的包下所有组件导入MainApplication所在的包
```

```java
@Import({AutoConfigurationImportSelector.class})  
/**
* 利用getAutoConfigurationEntry(annotaionMetadata)批量导入一些组件
* 调用getCandidateConfigurations(annotaionMetadata, attributes)获取所以需要导入的全类名
* loadSpringFactories得到所有组件
* 从META-INF/spring.factories加载文件，默认扫描所有系统此位置的文件
*/
```

**按需配置**

并不是所有的自动配置都会生效，启动时会加载所有配置，根据条件装配注解决定哪些配置生效