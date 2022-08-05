### 目录结构

- /bin 可执行文件
- /conf 日志log.properties、服务器server.xml、web应用web.xml配置文件
- /lib 依赖的jar包
- /logs 日志
- /temp
- /webapps 默认web应用
- /work java源码字节码

### 请求流程

HTTP服务器 => Tomcat服务器 => Servlet调用业务代码

### 架构

##### 连接器

![image-20220628215015105](E:\学习笔记\typora\img\image-20220628215015105.png)

coyota：支持NIO，由JDK的NIO2类库实现.

![image-20220629212428187](E:\学习笔记\typora\img\image-20220629212428187.png)

##### 容器

![image-20220629212739250](E:\学习笔记\typora\img\image-20220629212739250.png)

Context就是我们的Web Application。

Server下面可以有多个Service，一个Service下面可以有多个Connector，但只能有一个Container。一个Container只能有有一个Engine，但能有多个Host。一个Host可以有多个Context。

### 启动流程

![image-20220704211348150](E:\学习笔记\typora\img\image-20220704211348150.png)

此流程抽象到LifeCurcle中

### 请求流程

Mapper组件根据请求url定位到servlet

![image-20220709230809100](E:\学习笔记\typora\img\image-20220709230809100.png)