# 引言
&emsp;&emsp;HTTP 协议（HyperText Transfer Protocol，超文本传输协议），HTTP 基于 TCP/IP 通信协议实现客户端与服务端的通信标准，默认端口是 80，客户端发起一个 http 请求与服务端进行通信，请求所需的资源，如 html 文本、图片等，我们称这个客户端为**用户代理程序**（user agent），称这个服务器为**源服务器**（origin server）。<br/>
&emsp;&emsp;由 HTTP 客户端发起一个请求，创建一个到服务器指定端口（默认是 80 端口）的 TCP 连接。HTTP 服务器则在那个端口监听客户端的请求。一旦收到请求，服务器会向客户端返回一个状态，比如"HTTP/1.1 200 OK"，以及返回的内容，如请求的文件、错误消息、或者其它信息。如同人与人之间遵循规范的通信规则沟通对话一般。
## 版本特性
&emsp;&emsp;在这里只是大概说一些比较重要的版本特性，具体需要大家逐个细节去学习。
### HTTP/0.9
&emsp;&emsp;HTTP 协议的最初版本，请求方式只有 GET，最早版本的 HTTP 协议并没有复杂的需求，仅仅需要达到通信需求即可，HTTP0.9 也只支持 HTML 文件，所以常常用于学术交流等场景。<br/>
&emsp;&emsp;**请求方式**：GET
#### 特性
- HTTP0.9 只有请求行，没有 HTTP 请求头和 HTTP 请求体。
- HTTP 响应没有头信息
- 数据是以 ASCLL 字符流返回给客户端
### HTTP/1.0
&emsp;&emsp;请求行中引入了协议版本字段(http/1.0)；请求和响应必须包含头部消息，不在局限仅能请求 HTML 格式的资源,可以支持多种数据格式、缓存等功能。<br/>
&emsp;&emsp;**请求方式**：POST、HEAD、GET
#### 特性
HTTP1.0 中有许多特性，如下：
 - **灵活**：HTTP 允许传输任意类型的数据对象。正在传输的类型由 Content-Type 加以标记。
 - **简单快速**：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有 GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因而通信速度很快。
 - **性能**：支持对数据压缩后传输，大大提高传输性能。
 - **无连接**：因为 HTTP 是一个基于 TCP/IP 通信协议来传递数据的协议，在此版本中 HTTP 仅仅支持一次性连接，正如大家所熟知的，TCP 连接时需要三次握手，发送请求数据，最后四次挥手结束连接。当 HTTP 基于 TCP 发送一次连接请求，并得到服务端响应后，会立即断开连接，也称非持久连接，缺点是造成大量的建立连接消耗。
 - **无状态**：无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。
		
### HTTP/1.1
&emsp;&emsp;此版本最大的变化就是引入了持久连接，允许同时建立至多 6 个 TCP 连接，解决了非持久连接的缺点，TCP 连接默认不关闭，连接一次，可发送多个请求。<br/>
&emsp;&emsp;引入了管道机制，一个 TCP 连接中，多个请求在客户端可以同时发送，客户端不需要逐个发送请求，但服务器按照先来后到的顺序进行处理。<br/>
&emsp;&emsp;引入了 Cookie 机制、安全机制，引入了更多缓存策略，除了 HTTP1.0 的 Header 里的 If-Modified-Sincce,Expires 字段，新增了如：If-Match、If-None-Match、If-Unmodified-Since 等更多的缓存策略<br/>
&emsp;&emsp;HTTP1.1 新增了 24 个错误状态响应码<br/>
&emsp;&emsp;**请求方式**：POST、HEAD、GET、PUT、PATCH、OPTIONS、DELETE
#### 特性
 - **持久连接**：允许同时建立至多 6 个 TCP 连接，TCP 连接默认不关闭，连接一次，可发送多个请求。
 - **管道机制**：尝试使用管道机制，多个请求在客户端允许同时发送，但是服务器依旧使用先进先出特性依次响应。
 - **多域名**：引入了 Host 头部字段用以表示域名地址，可以使一个 IP 拥有多个域名，服务器可以根据不同 Host 值区别处理请求。
 - **分块传输**：服务器把数据拆分成若干个数据块，传输给客户端，最后以零长度的数据块作为结束标识，以此标志完成传输。
### HTTP/2
&emsp;&emsp;增加了双工模式，不仅客户端能够同时发送多个请求，服务端也能同时处理多个请求，解决了队头堵塞的问题。
#### 特性
 - **多路复用**：通过一个 TCP 连接即可并行发送多个请求及响应，不再局限于先进先出的特性
 - **优先级**：可以对请求打上优先级标志，服务器可对优先级高的请求优先处理，最高优先级：0；最低优先级：2 的 31 次方-1。
 - **服务端推送**：此功能让服务器变得更体贴了，服务器在遵守“请求-响应原则”及“同源策略”的前提下，在真正响应之前多推送一些资源给客户端，当然客户端可以拒绝服务器的热情，设置 SETTINGS 帧中的服务端流的值为 0 即可。
 - **头部压缩**：实现原理是客户端和服务端会跟踪和存储头部字段的键值对，利用一个头部表对应此键值对，每次传输携带此表即可，对于新增的字段放到头部表的尾部进行传输，是 HTTP 协议的头部信息更快速的传输。
### HTTP/3
&emsp;&emsp;HTTP3.0，也称作 HTTP over QUIC。HTTP/3 是基于 UDP 的安全可靠的 HTTP/2 协议，基于 UDP 实现了类似 TCP 的多路数据流，传输可靠性等功能，称为 QUIC 协议。<br/>
&emsp;&emsp;目前浏览器和服务器并没有对 HTTP/3 有比较完整的支持，同时系统对 UDP 优化并不是很好，所有部署 HTTP/3 还存在很多的挑战。
#### 特性
QUIC 协议功能有以下特性：
 - 实现了类似 TCP 的流量控制，具有数据可靠性，因为 UDP 协议提供面向事务的简单不可靠信息传送服务，它不提供报文到达确认、排序、及流量控制等功能，但是 UDP 协议有很多优点，如延迟小、数据传输效率高等特性，能够改善 TCP 协议的缺陷。
 - 继承了 TLS 加密功能。
 - 实现了 HTTP/2 中多路复用功能，同时解决了 2.0 的 TCP 中队头阻塞的问题。
 - 实现了快速握手功能，QUIC 可以使用 0-RTT 或者 1-RTT 来建立连接。
# HTTP 协议
&emsp;&emsp;其实 HTTP 协议请求并不复杂，意想一下，就和一个面向对象的对象一样，为了充分理解，直接上图

![HTTP](https://files.mdnice.com/user/22227/ced74cfc-8cd5-480a-ac31-dbe881004216.jpg)

&emsp;&emsp;HTTP 消息分为客户端向服务端发送请求（http 请求）及服务端做出响应（http 响应）<br/>
&emsp;&emsp;http 请求由三部分组成，分别是：请求行、消息报头、请求数据<br/>
&emsp;&emsp;HTTP 响应也是由三个部分组成，分别是：状态行、消息报头、响应数据

## 请求行
&emsp;&emsp;请求行以一个方法符号开头，以空格分开，后面跟着请求的 URI 和协议的版本，格式如下：<br/>
```plain
Method Request-URI HTTP-Version CRLF
```
&emsp;&emsp;其中 Method 表示请求方法；Request-URI 是一个统一资源标识符；HTTP-Version 表示请求的 HTTP 协议版本；CRLF 表示回车和换行（除了作为结尾的 CRLF 外，不允许出现单独的 CR 或 LF 字符）
```javascript
GET /main.html HTTP/1.1
```
### 请求方法
```plain
	GET    请求获取Request-URI所标识的资源
	POST    在Request-URI所标识的资源后附加新的数据
	HEAD    请求获取由Request-URI所标识的资源的响应消息报头
	PUT     请求服务器存储一个资源，并用Request-URI作为其标识
	DELETE  请求服务器删除Request-URI所标识的资源
	TRACE   请求服务器回送收到的请求信息，主要用于测试或诊断
	CONNECT 保留将来使用
	OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求
```
    
举例：<br/>
&emsp;&emsp;**GET 方法**：一般在在浏览器的地址栏中输入网址访问网页时，浏览器采用		GET 方法向服务器获取资源，这种方式参数会拼接到 URL 后面，不太安全，并且有长度限制。<br/>
&emsp;&emsp;**POST 方法**：要求被请求服务器接受附在请求后面的数据，常用于提交表单。这种方式提交表单会把参数放在 Body 中，相对比较安全。
其他方法就不一一举例了，需要深入了解，可自行搜索。
## URL
&emsp;&emsp;HTTP URL 是一种特殊类型的 URI，包含了用于查找某个资源的足够的信息，格式如下：
```javascript
http://host[':'port][path]
如：http://127.0.0.1:8080/system/index
```
&emsp;&emsp;`http://`表示通过 HTTP 协议来定位网络资源；`host`表示主机域名或 IP 地址；`port`表示一个端口号，为空则使用默认端口`80`；`path`表示请求资源的 URI，如果 path 为空，则会默认以`/`的 URI 作为 path 去请求。
## 状态行
状态行格式如下：
```plain
HTTPVersion StatusCode Reason-Phrase CRLF
```
&emsp;&emsp;其中，HTTPVersion 表示服务器 HTTP 协议的版本；StatusCode 表示服务器响应的状态代码；ReasonPhrase 表示状态代码的文本描述。<br/>
&emsp;&emsp;状态代码有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：
```plain
	1xx：指示信息--表示请求已接收，继续处理
	2xx：成功--表示请求已被成功接收、理解、接受
	3xx：重定向--要完成请求必须进行更进一步的操作
	4xx：客户端错误--请求有语法错误或请求无法实现
	5xx：服务器端错误--服务器未能实现合法的请求
```
常见的状态代码有
```plain
	200 OK // 请求成功
	304 Not Modified // 服务端已经执行了GET，但文件未变化，从缓存中返回的资源
	400 Bad Request  //客户端请求有语法错误，不能被服务器所理解
	401 Unauthorized //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用 
	403 Forbidden  //服务器收到请求，但是拒绝提供服务
	404 Not Found  //请求资源不存在，eg：输入了错误的URL
	500 Internal Server Error //服务器发生不可预期的错误
	503 Server Unavailable  //服务器当前不能处理客户端的请求，一段时间后可能恢复正常
```
## 消息报头之请求报头
&emsp;&emsp;HTTP 消息报头包括普通报头、请求报头、响应报头、实体报头，先说说请求报头到底是什么。<br/>
&emsp;&emsp;每一个消息报头域都是由`[name][': '][value]`组成，消息报头域的名字是大小写无关的,如：`content-type: application/json`
对于请求报头，直接上图

![Request](https://files.mdnice.com/user/22227/1b04cd90-3732-4bf9-8185-22d02b8ed9f5.jpg)

&emsp;&emsp;这种就是请求报头，可以按`F12`打开开发者工具，进入 network 中点开任意一个请求查看。<br/>
常见的请求报头如下：

|Header|描述|案例|
|--|--|--|
|Accept|指定客户端能够接收的内容类型|Accept: text/plain, text/html|
|Accept-Charset|浏览器可以接受的字符编码集。|Accept-Charset: iso-8859-5|
|Accept-Encoding|指定浏览器可以支持的 web 服务器返回内容压缩编码类型。|Accept-Encoding: compress, gzip|
|Accept-Language|浏览器可接受的语言|Accept-Language: en,zh|
|Accept-Ranges|可以请求网页实体的一个或者多个子范围字段|Accept-Ranges: bytes|
|Authorization|HTTP 授权的授权证书|Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==|
|Cache-Control|指定请求和响应遵循的缓存机制|Cache-Control: no-cache|
|Connection|表示是否需要持久连接。（HTTP 1.1 默认进行持久连接）|	Connection: close|
|Cookie|HTTP 请求发送时，会把保存在该请求域名下的所有 cookie 值一起发送给 web 服务器。|Cookie: $Version=1; Skin=new;|
|Content-Length|请求的内容长度|Content-Length: 348|
|Content-Type|请求的与实体对应的 MIME 信息|Content-Type: application/x-www-form-urlencoded|
|Date|请求发送的日期和时间|Date: Tue, 15 Nov 2010 08:12:31 GMT|
|Expect|请求的特定的服务器行为|Expect: 100-continue|
|From|发出请求的用户的 Email|From: user@email.com|
|Host|指定请求的服务器的域名和端口号|Host: www.baidu.com|
|If-Match|只有请求内容与实体相匹配才有效|If-Match: “737060cd8c284d8af7ad3082f209582d”|
|If-Modified-Since|如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回 304 代码|If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT|
|If-None-Match|如果内容未改变返回 304 代码，参数为服务器先前发送的 Etag，与服务器回应的 Etag 比较判断是否改变|If-None-Match: “737060cd8c284d8af7ad3082f209582d”|
|If-Range|如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为 Etag|If-Range: “737060cd8c284d8af7ad3082f209582d”|
|If-Unmodified-Since|只在实体在指定时间之后未被修改才请求成功|If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT|
|Max-Forwards|限制信息通过代理和网关传送的时间|Max-Forwards: 10|
|Pragma|用来包含实现特定的指令|Pragma: no-cache|
|Proxy-Authorization|连接到代理的授权证书|Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==|
|Range|只请求实体的一部分，指定范围|Range: bytes=500-999|
|Referer|先前网页的地址，当前请求网页紧随其后,即来路|Referer: https://www.baidu.com|
|TE|客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息|TE: trailers,deflate;q=0.5|
|Upgrade|向服务器指定某种传输协议以便服务器进行转换（如果支持）|Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11|
|User-Agent|User-Agent 的内容包含发出请求的用户信息|User-Agent: Mozilla/5.0 (Linux; X11)|
|Via|通知中间网关或代理服务器地址，通信协议|Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)|
|Warning|关于消息实体的警告信息|Warn: 199 Miscellaneous warning|

## 消息报头之响应报头
&emsp;&emsp;HTTP 消息报头包括普通报头、请求报头、响应报头、实体报头，接下来说说响应报头。<br/>
&emsp;&emsp;响应报头允许服务器传递不能放在状态行中的附加响应信息，以及关于服务器的信息和对 Request-URI 所标识的资源进行下一步访问的信息。
直接上图

![pesponse](https://files.mdnice.com/user/22227/6cb5123b-f7cf-4515-90a8-1502b01a3900.jpg)

常见的响应报头
|Header|描述|案例|
|--|--|--|
|Accept-Ranges|表明服务器是否支持指定范围请求及哪种类型的分段请求|Accept-Ranges: bytes|
|Age|从原始服务器到代理缓存形成的估算时间（以秒计，非负）|Age: 12|
|Allow|对某网络资源的有效的请求行为，不允许则返回 405|Allow: GET, HEAD|
|Cache-Control|告诉所有的缓存机制是否可以缓存及哪种类型|Cache-Control: no-cache|
|Content-Encoding|web 服务器支持的返回内容压缩编码类型。|Content-Encoding: gzip|
|Content-Language|响应体的语言|Content-Language: en,zh|
|Content-Length|响应体的长度|Content-Length: 348|
|Content-Location|请求资源可替代的备用的另一地址|Content-Location: /index.htm|
|Content-MD5|返回资源的 MD5 校验值|Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==|
|Content-Range|在整个返回体中本部分的字节位置|Content-Range: bytes 21010-47021/47022|
|Content-Type|返回内容的 MIME 类型|Content-Type: text/html; charset=utf-8|
|Date|原始服务器消息发出的时间|Date: Tue, 15 Nov 2010 08:12:31 GMT|
|ETag|请求变量的实体标签的当前值|ETag: “737060cd8c284d8af7ad3082f209582d”|
|Expires|响应过期的日期和时间|Expires: Thu, 01 Dec 2010 16:00:00 GMT|
|Last-Modified|请求资源的最后修改时间|Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT|
|Location|用来重定向接收方到非请求 URL 的位置来完成请求或标识新的资源|Location: https://www.baidu.com
|Pragma|包括实现特定的指令，它可应用到响应链上的任何接收方|Pragma: no-cache|
|Proxy-Authenticate|它指出认证方案和可应用到代理的该 URL 上的参数|Proxy-Authenticate: Basic|
|refresh|应用于重定向或一个新的资源被创造，在 5 秒之后重定向（由网景提出，被大部分浏览器支持）|Refresh: 5; url=https://www.baidu.com|
|Retry-After|如果实体暂时不可取，通知客户端在指定时间之后再次尝试|Retry-After: 120|
|Server|web 服务器软件名称|Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)|
|Set-Cookie|设置 Http Cookie|Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1|
|Trailer|指出头域在分块传输编码的尾部存在|Trailer: Max-Forwards|
|Transfer-Encoding|文件传输编码|Transfer-Encoding:chunked|
|Vary|告诉下游代理是使用缓存响应还是从原始服务器请求|Vary: *|
|Via|告知代理客户端响应是通过哪里发送的|Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)|
|Warning|警告实体可能存在的问题|Warning: 199 Miscellaneous warning|
WWW-Authenticate|表明客户端请求实体应该使用的授权方案|WWW-Authenticate: Basic|
