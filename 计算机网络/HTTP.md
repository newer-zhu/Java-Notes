## HTTP

**请求头格式**

![image-20220414111347421](E:\学习笔记\typora\img\image-20220414111347421.png)

**返回格式**

![image-20220414111422821](E:\学习笔记\typora\img\image-20220414111422821.png)

## request

**请求数据格式**

请求行：GET /login.html HTTP/1.1 

请求头：

Host	请求主机
User-Agent	浏览器访问服务器使用的浏览器版本信息(在服务器获取后，解决浏览器兼容性问题)
Accept	支持的文件格式
Accept-Language	支持的语言
Accept-Encoding	支持的压缩格式
Referer	告诉服务器，请求从哪里来。作用：1.防盗链2.统计工作
Connection	表示连接的状态(活着就可以复用)
Upgrade-Insecure-Requests	关于升级的信息
upgrade websocket

请求空行

请求体

## http1.0

浏览器每次请求都需要与服务器建立一个 TCP 连接，服务器处理完成后立即断开 TCP 连接,无连接

## http1.1

默认使用 `Connection:keep-alive`（**长连接**）,支持管线化，并行发送多个请求

## http2.0

二进制传输、Header压缩、多路复用一个TCP、服务器端推送

TCP的**队头阻塞**没有彻底解决（http2.0中，多个请求是跑在一个TCP管道中的，一旦丢包，TCP就要等待重传（丢失的包等待重新传输确认），从而阻塞该TCP连接中的所有请求）

## http3.0

**基于UDP协议的“QUIC”协议，解决队头阻塞**