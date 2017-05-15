---
title: Android 网络编程之 HTTP 协议
date: 2016-07-21 09:29:16
tags: 网络
categories: Android
---
# 前言
如今是网络时代，几乎每一个 Android 应用都会用到网络，虽然目前有很多开源框架辅助我们进行网络的开发，但是作为开发者，基本的原理还是需要了解的。本文主要的内容是 HTTP 协议，以下内容并非完全原创，部分收集整理自网络。

# HTTP协议
## HTTP简介
hypertext transfer protocol（超文本传输协议），TCP/IP 协议的一个应用层协议，用于定义 Web 浏览器与 Web 服务器之间交换数据的过程。客户端连上 Web 服务器后，若想获得 Web 服务器中的某个 Web 资源，需遵守一定的通讯格式，HTTP 协议用于定义客户端与 Web 服务器通迅的格式。
### 主要特点
1. 支持 C/S（客户/服务器）模式。
2. 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有 GET、HEAD、POST，每种方法规定了客户与服务器联系的类型不同。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因而通信速度很快。
3. 灵活：HTTP 允许传输任意类型的数据对象。正在传输的类型由 Content-Type 加以标记。
4. 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
5. 无状态：HTTP 协议是无状态协议，无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

### HTTP URL
一个 URL 示例  
 http://www.mysite.com:8080/path/test;id=1024?name=hello&age=18#student

- Scheme: http 	（**指定底层使用的协议，如 http，https，ftp**)
- host: www.mysite.com  （**服务器的ip地址或域名**）
- port: 8080（**默认端口是 80，可省略，若指定其他端口，需要指明**）
- path: /path/test  （**访问资源路径**）
- URL params: id=1024 （**用于指定特殊参数的可选项**）
- Query String: name=hello&age=18 （**查询参数**）
- Fragment: student （**信息片段，字符串，指定网络资源中的片段**）

### HTTP 和 Socket 的对比
HTTP 协议是** 无状态 **的，这里就涉及到 HTTP 连接和 Socket 连接的主要区别  

||HTTP|Socket|
|---|---|---|
|定义|所谓的短连接，即客户端向服务器发送一次请求,服务端响应后连接立即会断掉|所谓的短连接,即客户端向服务器发送一次请求，服务端响应后连接立即会断掉|
|优点|稳定，吞吐量大|传输数据时间短，性能高|
|缺点|实时通信不方便，并发问题|消耗带宽,开发量较大|
|应用场景|适用于大部分场景|用于 IM|

# 请求和响应
## 请求（Request）
### 请求信息
请求信息主要包括三个部分：请求行（Request line），请求头（http header），请求正文（body） 

- ** 请求行 **
主要包含请求种类，请求自资源路径，HTTP 协议版本  ，格式如下
> Method Request-URI Http-Version CRLF

其中 Method 表示请求方法；Request-URI 是一个统一资源标识符；HTTP-Version 表示请求的 HTTP 协议版本；CRLF 表示回车和换行（除了作为结尾的 CRLF 外，不允许出现单独的 CR 或 LF 字符）。

- ** 请求头 **
请求头里面有很多键值对，很多都是浏览器默认的，当然也可以自己定制，只需要和服务器约定好  
常用的有

|Header  | 解释|示例|
| --- | --- | --- |
| Accept | 指定客户端能接受的内容类型 | Accept：text/html |
| Accept-Charset |  浏览器可以接受的编码字符集| Accept-Charset: iso-8859-5 |
| Accept-Encoding |  指定浏览器可以支持的 Web 服务器返回内容压缩编码类型| Accept-Encoding: compress, gzip |
| Cache-Control | 指定请求和响应遵循的缓存机制|Cache-Control: no-cache，Public,Pricate|
|Connection|表示是否需要持久连接（Http1.1默认持久连接）|Connection: keep-alive,close|
|Cookie|请求时，把 Cookie 值发送给 Web 服务器|Cookie: $Version=1; Skin=new;|
|Content-Type|请求的与实体对应的 MIME 信息|Content-Type: application/x-www-form-urlencoded|
|If-Modified-Since	|如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回 304 代码|If-Modified-Since: Thur, 21 July 2016 13:43:31 GMT
|If-None-Match	|如果内容未改变返回 304 代码，参数为服务器先前发送的 Etag，与服务器回应的 Etag 比较判断是否改变	|If-None-Match: "39819hfiwuhfi"|

### 请求方法
所有请求方式如下：

- GET：请求获取Request-URI所标识的资源
- POST：在Request-URI所标识的资源后附加新的数据
- HEAD 请求获取由Request-URI所标识的资源的响应信息报头
- PUT：请求服务器存储一个资源，并用Request-URI作为其标识
- DELETE：请求服务器删除Request-URI所标识的资源
- TRACE：请求服务器回送收到的请求信息，主要用于测试或诊断
- CONNECT：保留将来使用
- OPTIONS：请求查询服务器的性能，或者查询与资源相关的选项

其中最常用的就是 GET 和 POST，同时也要了解一下 PUT 和 DELETE，这四种是最重要的，对应了数据库的增删查改，常用的 GET 和 POST 对比如下：

|GET|POST|
|---|---|
|请求信息放在 URL 后面|请求放在实体内容|
|数据容量不大于 2KB|无容量限制|
|安全性低，效率高|安全性高，效率低|
|一般用于查询数据|用于数据增删改|

从本质上来说，这两个请求方式没有区别，都是用来发送数据，但是由于 HTTP 的规定和浏览器/服务器的限制，导致他们在应用过程中体现出一些不同。 
详情可参考[99% 的人都理解错了 HTTP 中 GET 与 POST 的区别](http://mp.weixin.qq.com/s?__biz=MzI3NzIzMzg3Mw==&mid=100000054&idx=1&sn=71f6c214f3833d9ca20b9f7dcd9d33e4#rd)


## 响应（Response）
Response 消息中的第一行叫做状态行，包括三个部分：HTTP 协议版本号，状态码，状态消息
### 状态码
`Http1.1` 中定义了 5 类状态码

- 1XX  信息 - 表示请求已被成功接收，继续处理
- 2XX  成功 - 表示请求已被成功接收，理解，接受
- 3XX  重定向 - 要完成请求必须进行更进一步的处理
- 4XX  客户端错误 - 请求有语法错误或请求无法实现
- 5XX  服务器端错误 - 服务器未能实现合法的请求

常见的状态码：

- 200 OK — 请求被成功地完成，所请求的资源发送回客户端
- 302 Found — 重定向，新的 URL 会在 Response 中的 Location 中返回，浏览器将会使用新的URL发出新的 Request
- 304 Not Modified - 文档已经被缓存，直接从缓存调用
- 400 Bad Request - 客户端请求与语法错误，不能被服务器所理解 
- 403 Forbidden - 服务器收到请求，但是拒绝提供服务 
- 404 Not Found - 请求资源不存在
- 500 Internal Server Error - 服务器发生了不可预期的错误 
- 503 Server Unavailable - 服务器当前不能处理客户端的请求，一段时间后可能恢复正常

### 响应头

|Header  | 解释|示例|
| --- | --- | --- |
| Cache-Control | 指定请求和响应遵循的缓存机制|Cache-Control: no-cache，Public,Pricate|
|Content-Type	|返回内容的 MIME 类型	|Content-Type: text/html; charset=utf-8|
| ETag | 标签当前值，和 request header 的 If-None-Match 配合使用  | ETag: "9238ehd928" |
|Last-Modified	|请求资源的最后修改时间，配合 If-Modified-Since 使用	|Last-Modified: Tue, 15 Nov 2016 13:45:26 GMT|

更多关于请求头和响应头的内容请参考[Android Http 请求头与响应头的学习](http://www.kancloud.cn/kancloud/android-tutorial/87222)

# 总结
本文只是简单介绍了 Android 网络编程中需要了解的 HTTP 协议相关的知识，希望能够作为后面学习者的一个参考。若有不足之处，烦请指出

# 参考资料
- [Android 网络编程要学的东西与 Http 协议学习](http://www.kancloud.cn/kancloud/android-tutorial/87221)
- [HTTP 协议详解](https://www.zybuluo.com/yangfch3/note/167490)



