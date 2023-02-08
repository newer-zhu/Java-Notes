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

默认使用 `Connection:keep-alive`（**长连接**）,支持管线化，并行发送多个请求。增加了缓存和控制模块。存在对头阻塞问题。

## http2.0

二进制传输、Header压缩、多路复用一个TCP、服务器端推送

HTTP/2.0 解决队头阻塞的问题是采用了 stream 和分帧的方式。

HTTP/2.0将一个 TCP 连接切分成为多个 stream，每个 stream 都有自己的 stream id，这个 stream 可以是客户端发往服务端，也可以是服务端发往客户端。

HTTP/2.0 还能够将要传输的信息拆分为帧，并对它们进行二进制格式编码。也就是说，HTTP/2.0 会将 Header 头和 Data 数据分别进行拆分，而且拆分之后的二进制格式位于多个 stream 中。

TCP的**请求阻塞**没有彻底解决（http2.0中，多个请求是跑在一个TCP管道中的，一旦丢包，TCP就要等待重传（丢失的包等待重新传输确认），从而阻塞该TCP连接中的所有请求）

==假如有一个请求有三个 stream，其中 stream2 由于某些原因丢失了，那么 stream1 和 stream 2 的处理也会阻塞，只有收到重发的 stream2 之后，服务器才会再次进行处理。==

## http3.0

**基于UDP协议的“QUIC”协议，解决队头阻塞**

#### QUIC

1. 减少了 TCP 三次握手及 TLS 握手时间。
2. 改进的拥塞控制。
3. 避免队头阻塞的多路复用。
4. 连接迁移。
5. 前向冗余纠错。

"TCP+TLS"握手一共需要在客户端和服务端进行4个来回，一共需要4个RTT

QUIC提供了0-RTT和1-RTT的连接建立。