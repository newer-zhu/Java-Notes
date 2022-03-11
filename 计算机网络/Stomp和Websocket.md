STOMP(Simple Text-Orientated Messaging Protocol) 面向消息的简单文本协议

[WebSocket](https://so.csdn.net/so/search?q=WebSocket&spm=1001.2101.3001.7020)是一个消息架构，不强制使用任何特定的消息协议，它依赖于应用层解释消息的含义；

与处在应用层的[HTTP](https://so.csdn.net/so/search?q=HTTP&spm=1001.2101.3001.7020)不同，WebSocket处在TCP上非常薄的一层，会将字节流转换为文本/二进制消息，因此，对于实际应用来说，WebSocket的通信形式层级过低，因此，可以在 WebSocket 之上使用 STOMP协议，来为浏览器 和 server间的 通信增加适当的消息语义。

如何理解 STOMP 与 WebSocket 的关系：
\1) HTTP协议解决了 web 浏览器发起请求以及 web 服务器响应请求的细节，假设 HTTP 协议 并不存在，只能使用 TCP 套接字来 编写 web 应用，你可能认为这是一件疯狂的事情；
\2) 直接使用 WebSocket（SockJS） 就很类似于 使用 TCP 套接字来编写 web 应用，因为没有高层协议，就需要我们定义应用间所发送消息的语义，还需要确保连接的两端都能遵循这些语义；
\3) 同 HTTP 在 TCP 套接字上添加请求-响应模型层一样，STOMP 在 WebSocket 之上提供了一个基于帧的线路格式层，用来定义消息语义；

## STOMP帧

STOMP帧由命令，一个或多个头信息、一个空行及负载（文本或字节）所组成；

其中可用的COMMAND 包括：

> CONNECT、SEND、SUBSCRIBE、UNSUBSCRIBE、BEGIN、COMMIT、ABORT、ACK、NACK、DISCONNECT；

例：
发送消息

> SEND
> destination:/queue/trade
> content-type:application/json
> content-length:44
> {“action”:”BUY”,”ticker”:”MMM”,”shares”,44}^@

订阅消息

> SUBSCRIBE
> id:sub-1
> destination:/topic/price.stock.*
> ^@

服务器进行广播消息

> MESSAGE
> message-id:nxahklf6-1
> subscription:sub-1
> destination:/topic/price.stock.MMM
> {“ticker”:”MMM”,”price”:129.45}^@