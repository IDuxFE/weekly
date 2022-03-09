登录鉴权是业务中非常重要的一个环节，涉及到后续所有接口的请求权限和数据安全问题。虽然目前业界已经有很成熟的解决方案，但是肯定还是有些小伙伴知其然，不知其所以然。为了探索其中的实现原理，今天笔者就来简单的分析一波，用图画的方式配上少许文字，希望您能理解。

**PS：本篇文章不讲太多原理性的内容**

![前端登录鉴权方案.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f066eab952c048e781b723fd85fea80b~tplv-k3u1fbpfcp-watermark.image?)

以上是我总结的四种登录鉴权方案，废话不多话，直接上图吧

#### （1）纯cookie验证

![Untitled-2022-01-07-1013.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90f9397685184b4299155377f943d518~tplv-k3u1fbpfcp-watermark.image?)

##### 步骤分解

1.  浏览器首次登录，服务器收到请求后去数据库校验用户名及密码
1.  校验通过后，从数据库查询对应的用户信息
1.  把非重要信息（如密码，邮箱等）通过响应体`set-cookie`存入到浏览器中
1.  浏览器下次请求接口，自动携带`cookie`信息在请求体中，其中包含了用户的信息（如`username`）
1.  服务器拿到请求体中的`cookie`，如果包含`username`等用户信息，则校验通过

##### 细节点

服务器在响应体设置`cookie`时，可以设置过期时间（`expires`）以及哪些作用范围（`path`）

```
let d = new Date()
d.setTime(d.getTime() + 15 * 60 * 1000)
res.setHeader('Set-Cookie', `username=zhangsan; path=/; expires=${d.toGMTString()}`)
```

浏览器可以直接修改`cookie`的，在控制台或者`Application`操作

```
document.cookie='name=123'
```

为了防止通过`js`脚本修改`cookie`，可以增加`httpOnly`的限制，但是通过`Application`仍然可以修改

```
res.setHeader('Set-Cookie', `username=zhangsan; path=/; expires=${d.toGMTString()}; httpOnly`)
```

##### 优点

操作简单

##### 缺点

1.  服务器压力大，每次请求都需要去数据库校验`cookie`中的信息，数据库查询次数过高
1.  安全性低，客户端随时可以篡改`cookie`
1.  `cookie`大小在`4kb`，且只能存储字符串，存储限制较大
1.  `cookie`跨域无法共享

##### 总结

纯`cookie`的校验几乎是淘汰的，因为性能低，安全性差，但是为其他校验方式奠定了基础

#### （2）session + cookie

![Untitled-2022-01-07-2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c205db7e23e649249e70d751a7cc73dc~tplv-k3u1fbpfcp-watermark.image?)

##### 步骤分解

1.  浏览器首次登录，服务器收到请求后去数据库校验用户名及密码
1.  校验通过后，从数据库查询对应的用户信息
1.  在服务器进程中创建`session`，把用户信息保存起来
1.  设置响应体（`Set-Cookie`），把`sessionId`（一般是`userId`，因为`userId`比直接暴露`username`更安全）返回
1.  浏览器下次请求接口，自动携带`cookie`信息在请求体中，其中包含了`sessionId`
1.  服务器拿到请求体中的`sessionId`，判断`sessionId`对应的是哪个用户
1.  查询到对应用户后，执行后续操作。反之，权限校验失败

##### 细节点

1.  `session`是服务端存储，而`sessionStorage`和`cookie`一样，属于客户端存储
1.  `session`是保存在服务端进程中的，关闭服务端进程，`session`则无法获取了
1.  数据库是和服务端进程并非同一个东西，例如`nodejs`的`server`进程与`mysql`
1.  `session`可以存储任意类型的数据，且大小比`cookie`大得多

##### 优点

1.  安全性较高，客户端拿到的只是`sessionId`，没有具体的用户信息
1.  用户信息保存在服务器进程中，减少了查询数据库的次数
1.  可以设置`session`的过期时间，不受`cookie`过期时间的影响
1.  存储的数据类型不受限制

##### 缺点

1.  占用服务器内存资源，用户量很大时，内容消耗不起
1.  正常情况下，`session`在不同的服务器进程中无法共享，比如A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录。如果要实现，这要求每个进程都能共享`session`（可以通过`session`持久化的方式解决）

##### session持久化

上面提到的A网站和B网站的例子，其实就是我们经常听到的`单点登录`。解决不同进程共享`session`，业界也有很多方案，这里不过多赘述，比如利用`Redis`这样的工具实现`session`共享等。解决`session`占用服务器内存，可以扩展集群等。

#### （3）token

![Untitled-2022-01-07-1013-3.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5090b39b41454a7a88251509559b4436~tplv-k3u1fbpfcp-watermark.image?)
##### 步骤分解

1.  浏览器首次登录，服务器收到请求后去数据库校验用户名及密码
1.  校验通过后，从数据库查询对应的用户信息
1.  服务器将用户信息通过密钥生成令牌（`Access Token`），并返回给浏览器
1.  浏览器拿到令牌之后，可以保存到`cookie`或者`localStorage`、`sessionStorage`中
1.  浏览器下次请求接口，将`Access Token`放到请求头`header`中
1.  服务端拿到请求头的`token`，然后通过密钥解密，校验通过后执行后续操作。反之，权限校验失败

##### 细节分析

1.  基于`token`的认证方式是属于`无状态的`（服务器不会保存用户信息），而`session`是有状态的
1.  `token`认证完全由应用管理，可以避开同源策略（只要携带`token`，服务器可以校验通过，就忽略同源策略）
1.  `token`除了可以放在请求头，还可以在`get`请求中，比如：`http://xxxxx.com/getData?token=xxxx`，也可以放在`post`请求中，与`postData`一起发送到服务器
1.  `token`加密常见的是使用[HMACSHA256 类 (System.Security.Cryptography) | Microsoft Docs](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.microsoft.com%2Fzh-cn%2Fdotnet%2Fapi%2Fsystem.security.cryptography.hmacsha256%3Fredirectedfrom%3DMSDN%26view%3Dnetframework-4.8)

##### 优点

1.  `无状态`可以减轻服务器压力，减少频繁查询数据库
1.  没有同源策略的限制，方便第三方平台或者开发时的接口调用
1.  安全性较高，`token`的解密密钥只有服务端知道，即使客户端暴露出来，别人也无法解密

##### 缺点

1.  `token`过期时间较短，往往需要配合`refresh token`一起使用，`refresh token`是在`access token`过期时用来重新获取`token`的
1.  `refresh token`也有过期时间，且一般存储在数据库中，虽然不需要向 `session`一样一直保持在内存中以应对大量的请求，但也会增加一定次数的数据库查询

##### 总结

`token`提供的是一种令牌，授权，是让客户端有权限访问服务端资源的。而`session`是一种状态，如保持登录状态（记住密码功能）。具体用哪个需要结合业务，**如果你需要实现有状态的会话，可以使用 `Session`来在服务器端保存状态。如果你的用户数据可能需要和第三方共享，或者允许第三方调用 `api`接口，则可以使用 `token`。**

#### （4）JWT

![Untitled-2022-01-07-1013-4.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b77f1047a6a49c28113601ba14db6cc~tplv-k3u1fbpfcp-watermark.image?)

##### 步骤分解

1.  浏览器首次登录，服务器收到请求后去数据库校验用户名及密码
1.  校验通过后，从数据库查询对应的用户信息
1.  服务器将用户信息生成`jwt`，并返回给客户端（什么是`jwt`，可以看一下[阮一峰老师的---JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)，这里就不过多造次了）
1.  浏览器拿到`jwt`之后，可以保存到`cookie`或者`localStorage`、`sessionStorage`中
1.  浏览器下次请求接口，将`jwt`放到请求头`header`中（默认格式`Authorization: Bearer jwt`）
1.  服务端检查`jwt`的签名信息，从`jwt`中获取用户信息，并执行后续操作。反之，权限校验失败

##### 细节点

`jwt`默认是不加密的，本质是由（`Header.Payload.Signature`）组成。如需加密，可以和`access token`一样使用密钥加密

##### 优点

1.  安全性高，可以说是目前登录鉴权最优的方案，大部分公司都采用该方案
1.  解决服务器压力，减少服务器查询
1.  服务端也是`无状态`的
1.  不需要查询数据库，即可获取用户信息，因为`jwt`包含了用户信息和加密的数据。

##### 缺点

`jwt`不加密的情况下，不能将秘密数据写入 `jwt`

##### 总结

`jwt`和`token`的比较：

相同点：

-   都是访问资源的令牌
-   都可以记录用户的信息
-   都是使服务端无状态化

不同点：

-   `token`需要验证是否过期，正常情况下，需要配合`refresh token`一起使用，相比于`jwt`比较麻烦。而且`token`一般是需要存储在`redis`或者数据库中，服务器拿到`token`信息需要去校验真伪，所以`jwt`的优势就出来了

#### 参考

1.  [阮一峰老师的---JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
1.  [傻傻分不清之 Cookie、Session、Token、JWT](https://juejin.cn/post/6844904034181070861#heading-18)