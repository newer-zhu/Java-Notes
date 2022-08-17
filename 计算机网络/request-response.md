## request

**请求数据格式**

请求行：GET /login.html HTTP/1.1 

请求头

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

**重定向：**两次请求，不可以使用request对象共享数据，能访问其他服务器资源，地址栏发生变化
redirect：需要添加虚拟目录

**转发**：一次请求，可以使用request对象共享数据，只能访问本服务器，地址栏路径不变
forward：不需要添加虚拟目录

