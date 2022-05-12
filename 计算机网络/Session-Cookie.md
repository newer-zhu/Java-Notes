## Cookie

Cookie 常用来标记用户或对用户授权，浏览器发送出Cookkie之后可能被劫持用于非法行为。

**跨站请求伪造**（英语：Cross-site request forgery），也被称为 one-click attack 或者 session riding，通常缩写为 CSRF 或者 XSRF， 是一种挟制用户在当前已登录的 Web 应用程序上执行非本意的操作的攻击方法。跟跨网站脚本（XSS）相比，XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

**单个cookie限制4kb左右，数量限制20个，默认情况下cookie不能共享**

## Session

Session依赖于Cookie，但也可依赖于其他实现方式，如URL重写

第一次获取session没有cookie，在内存中创建session对象

响应给浏览器一个set-cookie头，值是session的id，浏览器第二次请求时又利用这个值找到同一个session

session默认失效时间为30min

session可以储存任意类型，任意大小的数据，可以用于储存一次会话的多次请求

**session钝化**：服务器关闭前，将session对象序列化到硬盘上。

**session活化**：tomcat自动活化session，两个session的id相同

## Token

- **Header 头部信息**：记录了使用的加密算法信息；
- **Payload 净荷信息**：记录了用户信息和过期时间等；
- **Signature 签名信息**：根据 header 中的加密算法和 payload 中的用户信息以及密钥key来生成，是服务端验证的重要依据。

服务端收到 token 后剥离出 header 和 payload 获取算法、用户、过期时间等信息，然后根据自己的加密密钥来生成 sign，并与客户端传来的 sign 进行一致性对比，来确定客户端的身份合法性。