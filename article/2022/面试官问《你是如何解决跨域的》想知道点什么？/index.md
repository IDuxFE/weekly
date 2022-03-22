这个问题已经老生常谈了，为什么面试官都喜欢问这个问题，因为可以拓展很多知识点，例如同源策略，有哪些解决跨域的办法，各有什么优缺点等，下面笔者将一一道来，希望能让你攻克这一系列问题，让面试官眼前一亮，早日拿到心仪offer。

#### 一、考点一，什么是跨域？

从字面上看，跨域就是跨域名，但是真正的跨域的并没有那么简单，因为浏览器存在**同源策略**，是浏览器厂商设置的一个重要的安全策，看一下[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)对浏览器同源策略的介绍：

> **同源策略**是一个重要的安全策略，它用于限制一个[origin](https://developer.mozilla.org/zh-CN/docs/Glossary/Origin)的文档或者它加载的脚本如何能与另一个源的资源进行交互。它能帮助阻隔恶意文档，减少可能被攻击的媒介。

如果缺少同源策略，浏览器很容易受到`XSS`、`CSRF`等攻击。

同源的定义：

只有协议、域名、端口都相同的情况下，才属于同源，如图所示，阴影区域必须都保持一致才属于同源。反之，如果三者有一个不一致，就会产生跨域。
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d61c09ccfa1a4533b2544f4dbd87b0fc~tplv-k3u1fbpfcp-watermark.image?)
下面列举几个场景：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2df39743d5a0425395a09d0c87aa8651~tplv-k3u1fbpfcp-watermark.image?)
需要注意的是：

1.  默认情况下，`http`协议可以省略端口`80`，`https`协议可以省略端口`443`，例如`http://www.example.com:80`和`http://www.example.com`是属于同源的
2.  如果是协议和端口造成的跨域问题，"前台"是无能为力的
3.  在跨域问题上，仅仅是通过"URL的首部"来识别，而不会根据域名对应的`IP`地址来判断，只要`URL`的首部（协议，域名，端口）一致，则属于同源

#### 二、考点二，服务器能否收到跨域的请求？

这里很容易搞混，既然跨域了，那发送的请求应该会被浏览器给砍掉，不会到达服务器端，实则不然。**跨域是可以正常发起请求的，服务器端能够收到请求并且正确返回结果，只是被浏览器拦截了**，用一张图说明一下：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2f9c48ae8f84152a6885764a51d9df1~tplv-k3u1fbpfcp-watermark.image?)

要永远记住一个原则：**同源策略只存在浏览器端**，服务器是没有跨域问题的，甚至用`postman`等工具也不会出现跨域问题


#### 三、考点三，有哪些解决跨域的方式？
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9f725f33a844b17839634b0fa403517~tplv-k3u1fbpfcp-watermark.image?)

这里整理了10钟解决跨域的方案，下面一一介绍

##### （1）CORS

虽说浏览器会默认拦截服务端返回的跨域请求数据，但是也是有办法让浏览器把这个拦截关掉的，那就是使用**CORS**。**CORS**是一个`W3C`标准，全称是"跨域资源共享"，它允许浏览器向跨域服务器，发出`XMLHttpRequest`请求，从而克服同源的限制。

`CORS`需要浏览器和服务器同时支持，目前浏览器都支持该功能，但是有版本限制：
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70b9e13eabdd4bb1a04aa0fbb7df292c~tplv-k3u1fbpfcp-watermark.image?)
`CORS`实现的本质，就是在浏览器请求中，自动添加一些附加的头信息。整个`CORS`通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，`CORS`通信与同源的`Ajax`通信没有差别，代码完全一样，对于用户来说，也是无感知的。

因此，实现`CORS`通信的关键是服务器。只要服务器实现了`CORS`接口，就可以跨源通信。

那么服务器如何实现`CORS`呢，前面有说过，实质是是给请求上添加一些头信息。服务器会根据请求是**简单请求**还是**非简单请求**来设置不同的头信息，什么是简单请求和非简单请求，这里可以看一下阮一峰老师的[跨域资源共享CORS详解](https://www.ruanyifeng.com/blog/2016/04/cors.html)，这里不过多赘述。其中最关键的是设置响应头：

```
Access-Control-Allow-Origin: <origin> | *
```

`origin`参数的值指定了**允许该资源的外域URL**，例如在`Node`中配置：

```
res.setHeader('Access-Control-Allow-Origin', 'http://www.abc.com')
```

这样来自`http://www.abc.com`的请求将被允许跨域访问，如果设置为*号，则允许来自任何域的请求：

```
res.setHeader('Access-Control-Allow-Origin', '*')
```

如果是是使用`Node` + `Express`开发服务端，推荐使用`npm`的包：[cors](https://www.npmjs.com/package/cors)，使用方式：

```
//安装包
npm install cors
​
const express = require('express')
const app = express()
​
//引入cors
const cors = require('cors')
​
//使用
app.use(cors())
```

使用第三方包的话，会更加方便的帮我们自动根据**简单请求**还是**非简单请求**来设置头信息，从而实现跨域通信。

##### （2）JSONP

和`CORS`一样，`JSONP`也是常见的一种解决跨域的方式，但是存在一个缺陷，`JSONP`只支持`GET`请求，而`CORS`支持所有类型的`HTTP`请求。但是它的浏览器兼容性更好，支持较低版本的浏览器。

我们都知道，`link`和`script`标签是本身就支持跨域通信的，例如：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/4.6.1/css/bootstrap.css">
    <title>Document</title>
</head>
<body>
    
</body>
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js"></script>
<script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/4.6.1/js/bootstrap.js"></script>
</html>
```

利用这个特点，就产生了`JSONP`这种跨域通信的方式，它的基本思想是，网页通过添加一个`<script>`元素，向服务器请求数据，这种做法不受同源政策限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。

首先，网页动态插入`<script>`元素，由它向跨源网址发出请求：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        function addScriptTag(src) {
          var script = document.createElement('script');
          script.setAttribute("type","text/javascript");
          script.src = src;
          document.body.appendChild(script);
        }
​
        window.onload = function () {
          addScriptTag('http://www.abc.com?callback=getData');
        }
​
        function getData(data) {
          console.log('data: ' + data);
        };
    </script>
</body>
</html>
```

上面代码通过动态添加`<script>`元素，向服务器`http://www.abc.com`发出请求。注意，该请求的查询字符串有一个`callback`参数，用来指定回调函数的名字，这对于`JSONP`是必需的。

服务器收到这个请求以后，会将数据放在回调函数的参数位置返回，例如使用`Node`时：

```
const http = require('http')
​
const server = http.createServer((req, res) => {
    if (req.query && req.query.callback) {
        //需要的数据
        const data = {
            id: 1,
            name: '张三',
            age: 18
        }
        const str =  req.query.callback + '(' + JSON.stringify(data) + ')'
        res.end(str)
    }
})
​
server.listen(80, () => {
    console.log('server running at http://www.abc.com')
})
```

由于`<script>`元素请求的脚本，直接作为代码运行。这时，只要浏览器定义了`getData`函数，该函数就会立即调用，从而拿到服务器的数据。

##### （3）Node代理

`Node`代理解决跨域，也是目前框架层面常用的方式。因为**同源策略**只存在浏览器端，服务器不受限制，所以可以在服务器端中通过**代理**的方式，获取响应数据后返回给浏览器端，用一张图说明：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0c450e283384596af9d76c7b4010566~tplv-k3u1fbpfcp-watermark.image?)
###### 3.1 在node中实现

```
//server1 ---http://www.abc.com
​
const http = require('http')
​
const server = http.createServer((req, res) => {
    //因为server1直接和浏览器通信，需要设置cors需要的首部字段
    req.setHeader({
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': '*',
        'Access-Control-Allow-Headers': 'Content-Type'
    })    
    
    //转发给server2服务器
    http.request({
        host: 'http://www.example.com',
        path: '/getDaata',
        port: 80,
        method: req.method,
        headers: req.headers
    }, request => {
        let body = '';
        request.on('data', (chunk) => {
            body += chunk;
        }).on('end', () => {
            res.end(body);
        })
    })
    
})
​
server.listen(80, () => {
    console.log('server running at http://www.abc.com')
})
```

```
//server2 ---http://www.example.com
​
const http = require('http')
​
const server = http.createServer((req, res) => {
  const data = {
      id: 1,
      name: '张三',
      age: 18
  }
  //防止中文乱码
  res.setHeader('Content-type', 'text/html: charset=utf-8')
  res.end(JSON.stringify(data))
})
​
server.listen(80, () => {
  console.log('server running at http://www.example.com')
})
```

###### 3.2 在webpack中实现

```
//webpack.config.js
​
module.exports = {
    ....
    devServer: {
        port: 80,
        proxy: {
          "/api": {
            target: "http://www.example.com"
          }
        }
    }
}
```

详情看：[dev-server](https://webpack.docschina.org/configuration/dev-server/)

###### 3.3 在vue-cli中实现

```
//vue.config.js
​
module.exports = {
    devServer: {
        proxy: {
          '/api': {
             target: 'http://www.example.com',
              changeOrigin: true,
              pathRewrite: {
                 '^/api': ''
              }
          }
        }
    }
}
```

详情看：[devserver-proxy](https://cli.vuejs.org/zh/config/#devserver-proxy)

类似通过`vue-cli`脚手架实现跨域的方式还有很多，这里不过多赘述。

##### （4）Nginx反向代理

这里不花大篇幅介绍`Nginx`反向代理，因为还涉及**负债均衡**等概念，还是老样子，用一张图解释：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a44868786c434a33b519acb9b3ac6032~tplv-k3u1fbpfcp-watermark.image?)

浏览器请求反向代理服务器想要获取资源，代理服务器就会去该地址中所有服务器中去寻找一个空闲的服务器为你响应，用于均衡每台服务器的负载率。使用`Nginx`反向代理实际上是对浏览器的一种"哄骗"，让它认为自己访问到的是同域，实际上服务器作了"调包"。直接上代码：

首先安装`nginx`，详细安装教程可以看：[nginx安装教程](https://blog.csdn.net/diaojw090/article/details/89135073)

然后，修改`nginx`目录下的`nginx.conf`：

```
//nginx.conf
​
server {
    # 监听80端口号
    listen 80;
​
    # 监听访问的域名
    server_name http://www.abc.com;
​
    # 根据访问路径配置
    location / {
        # 把请求转发到 http://www.abc.com：8080
        proxy_pass http://www.abc.com：8080;
​
        # 兼容websocket
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
​
```

最后通过命令行`nginx -s reload`重新启动`nginx`即可。

**Nginx**相较于**CORS**，没有浏览器版本的限制，同时不会影响服务器的性能。

##### （5）Websocket

`WebSocket`是一种通信协议，使用`ws://`（非加密）和`wss://`（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。

它可以在网络浏览器和服务器之间建立“套接字”连接。简单地说：客户端和服务器之间存在持久的连接，而且双方都可以随时开始发送数据。详细教程可以看：

[WebSocket简介：将套接字引入网络](https://www.html5rocks.com/zh/tutorials/websockets/basics/)

这个没什么过多解释，直接上代码吧：

前端：

```
const socket = new WebSocket('ws://www.abc.com')
socket.onopen = () => {
    socket.send('发送信息...')
}
socket.onmessage = (e) => {
    console.log(e.data) //获取数据
}
```

服务端：

```
const WebSocket = require("ws");
const server = new WebSocket.Server({ port: 80 });
server.on("connection", socket => {  
    socket.on("message", data => {    
        socket.send(data);  
    });
});
```

##### （6）window.postMessage

以下是[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)对于`postMessage`介绍：

> **window.postMessage()** 方法可以安全地实现跨源通信。通常，对于两个不同页面的脚本，只有当执行它们的页面位于具有相同的协议（通常为https），端口号（443为https的默认值），以及主机 (两个页面的模数 [`Document.domain`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/domain)设置为相同的值) 时，这两个脚本才能相互通信。**window.postMessage()** 方法提供了一种受控机制来规避此限制，只要正确的使用，这种方法就很安全。

举例来说，父窗口`http://aaa.com`向子窗口`http://bbb.com`发消息，调用`postMessage`方法就可以了。

```
const popup = window.open('http://bbb.com', 'title')
popup.postMessage('你好，bbb!', 'http://bbb.com')
```

`postMessage`方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（`origin`），即"协议 + 域名 + 端口"。也可以设为`*`，表示不限制域名，向所有窗口发送。

子窗口向父窗口发送消息的写法类似

```
window.opener.postMessage('你好，aaa!', 'http://aaa.com')
```

父窗口和子窗口都可以通过`message`事件，监听对方的消息

```
window.addEventListener('message', function(e) {
  console.log(e.data) //消息内容
  console.log(e.origin) //消息发向的网址
  console.log(e.source) //发送消息的窗口
},false)
```

##### （7）document.domain + iframe

这个方式只能用于**协议，端口，主域名（二级域名）都相同，子域名（一级域名）不同**的情况下，例如：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe87a847e66d483e9edb01955fe271de~tplv-k3u1fbpfcp-watermark.image?)

实现原理：两个页面都通过设置`document.domain`为基础主域，就实现了同域:

```
<!-- http://a.abc.com:8080/index.html -->
​
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    我是http://a.abc.com:8080/index.html
    <iframe src="http://b.abc.com:8080/index.html" frameborder="0" onload="load()" id="iframe"></iframe>
    <script>
        document.domain = 'abc.com'
        function load() {
            console.log(iframe.contentWindow.a) //100
        }
    </script>
</body>
</html>
```

```
<!-- http://b.abc.com:8080/index.html -->
​
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    我是http://b.abc.com:8080/index.html
    <script>
        document.domain = 'abc.com'
        let a = 100
    </script>
</body>
</html>
```

##### （8）window.location.hash + iframe

因为`hash`值得改变，页面是不会重新刷新的，利用这个特点，就可以实现通信。

父窗口可以把信息，写入子窗口的片段标识符

```
const src = 'http://www.abc.com/index.html#' + data
document.getElementById('iframe').src = src
```

子窗口通过监听`hashchange`事件得到通知

```
window.onhashchange = checkMessage
​
function checkMessage() {
  let message = window.location.hash
  // ...
}
```

同样的，子窗口也可以改变父窗口的片段标识符

```
parent.location.href= target + "#" + hash
```

##### （9）window.name + iframe

`window.name`属性有个特征：`name`值在不同的页面（甚至不同域名）加载后依旧存在，并且可以支持非常长的 `name`值（`2MB`）。

比如：

```
//www.aaa.com
​
window.name = 'My name is aaa!!!'
​
setTimeout(function(){
    window.location.href = "http://www.bbb.com"
},1000)
```

跳转后：

```
//www.bbb.com
​
console.log(window.name) //My name is aaa!!!
```

`http://www.aaa.com/index.html`要访问`http://www.bbb.com/index.html`，可以借助一个中间代理界面`http://www.aaa.com/other.html`

```
<!-- http://www.aaa.com/index.html -->
​
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    我是http://www.aaa.com/index.html
    <iframe src="http://www.aaa.com/other.html" frameborder="0" onload="load()" id="iframe"></iframe>
    <script>
        let first = true;
        // onload事件会触发2次，第1次加载跨域页，并留存数据于window.name
        function load() {
            if(first) {
                iframe.src = "http://www.bbb.com/index.html"
                first = false
            } else {
                console.log(iframe.contentWindow.name) //我是bbb  
            }
        }
    </script>
</body>
</html>
```

```
<!-- http://www.aaa.com/other.html -->
​
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    我是http://www.aaa.com/other.html
</body>
</html>
```

```
<!-- http://www.bbb.com/index.html -->
​
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    我是http://www.bbb.com/index.html
    <script>
        window.name = '我是bbb'
    </script>
</body>
</html>
```

通过 `iframe`的 `src`属性由外域转向本地域，跨域数据即由 `iframe` 的 `window.name` 从外域传递到本地域。这个就巧妙地绕过了浏览器的跨域访问限制，但同时它又是安全操作，这种方法与 `document.domain` 方法相比，放宽了域名后缀要相同的限制。

至于为什么要通过一个中间代理界面，这个还是因为同源策略的问题，避免报错！不信的话可以自己尝试。

##### （10）修改浏览器安全配置

**这个方式不推荐！！！**

**这个方式不推荐！！！**

**这个方式不推荐！！！**

毕竟浏览器的安全配置是为了防止攻击，保护信息设立的，轻易不要关闭。但是有些开发者本地调试方便也会用到这个"奇葩"的办法。

以谷歌浏览器为例

###### Window
```
C:\Program Files\Google\Chrome\Application\chrome.exe --args --disable-web-security
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd904d8cf473461d8b68ae00d6b73f4f~tplv-k3u1fbpfcp-watermark.image?)
###### Mac

```
open -a "Google Chrome" --args --disable-web-security  --user-data-dir
```

###### Linux

```
chromium-browser --disable-web-security  
```

#### 四、总结

上面总结了10钟解决跨域的方式，应该可以说涵盖了所有解决跨域的方式。其中使用哪一种方式应当结合业务场景，我个人推荐按以下优先级来做出选择：

1.  首先推荐使用**CORS**的方式解决跨域，因为无需多余的服务器，且在前台无感知
2.  **Node代理**和**Nginx反向代理**根据服务器条件做选择，一般前端用前者较多（脚手架方式等）
3.  涉及**iframe**的，优先使用**postMessage**，操作更简单

#### 五、参考

1.  [浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
2.  [CORS](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)
3.  [浏览器同源政策及其规避方法](https://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)
4.  [跨域资源共享 CORS 详解](https://www.ruanyifeng.com/blog/2016/04/cors.html)
5.  [10种跨域解决方案（附终极大招）](https://juejin.cn/post/6844904126246027278#heading-43)
6.  [九种跨域方式实现原理（完整版）](https://juejin.cn/post/6844903767226351623)

