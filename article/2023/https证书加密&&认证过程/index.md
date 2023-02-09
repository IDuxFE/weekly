# https证书加密&&认证过程

- 摘自 [极客时间-浏览器工作原理与实践](https://time.geekbang.org/column/article/180213)

## 背景

- 鉴于 HTTP 的明文传输的特性，数据在传输过程中的每一个环节，数据都有可能被窃取或者篡改，如下图; 当我们使用 HTTP 传输的内容很容易被中间人窃取、伪造和篡改；具体来说就是HTTP 数据提交给 TCP 层中间经过的每个环节都有可能被黑客抓取或者篡改。


![中间人攻击.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df21d2b8241840ec9add96c75e4023d0~tplv-k3u1fbpfcp-watermark.image?)

- HTTP 的明文传输的安全性问题限制网上购物、在线转账等一系列场景应用，有加密的 HTTPS 就应运而生


- HTTPS 并非是一个新的协议，通常 HTTP 直接和 TCP 通信，HTTPS 则先和安全层通信，然后安全层再和 TCP 层通信; 安全层有两个主要的职责：**对发起 HTTP 请求的数据进行加密操作和对接收到 HTTP 的内容进行解密操作**。网络层结构如下

![HTTP_VS_HTTPS.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/faf00110dab44e2a956f13535798f5ec~tplv-k3u1fbpfcp-watermark.image?)

## 加密方案的选型

> 名词解释：

- 加密套件： 加密套件指明了SSL握手阶段和通信阶段所应该采用的各种算法。这些算法包括：认证算法、密钥交换算法、对称算法和摘要算法等。(https://www.openssl.net.cn/docs/215.html)

![密码套件.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5166e9e55eda4136a1d6bfa43ac1ec39~tplv-k3u1fbpfcp-watermark.image?)

### 对称加密

在两台电脑上加解密同一个文件，我们至少需要知道加解密方式和密钥，因此，在 HTTPS 发送数据之前，浏览器和服务器之间需要协商加密方式和密钥，过程如下所示：

![对称加密实现.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce75d0cde7b249d78748b2fa78a20716~tplv-k3u1fbpfcp-watermark.image?)

> **过程**：HTTPS 首先要协商加解密方式，这个过程就是 HTTPS 建立安全连接的过程。为了让加密的密钥更加难以破解，我们让服务器和客户端同时决定密钥；
- 浏览器发送它所支持的加密套件列表和一个随机数 client-random，这里的加密套件是指加密的方法，加密套件列表就是指浏览器能支持多少种加密方法列表。
- 服务器会从加密套件列表中选取一个加密套件，然后还会生成一个随机数 service-random，并将 service-random 和加密套件列表返回给浏览器。
- 最后浏览器和服务器分别返回确认消息。
- 浏览器端和服务器端都有相同的 client-random 和 service-random 了，然后它们再使用相同的方法将 client-random 和 service-random 混合起来生成一个密钥 master secret，有了密钥 master secret 和加密套件之后，双方就可以进行数据的加密传输了

> **存在的问题**：传输 client-random 和 service-random 的过程却是明文的，黑客可以拿到随机数合成密钥，数据还是可以破解


### 非对称加密

服务器会将其中的一个密钥通过明文的形式发送给浏览器，我们把这个密钥称为公钥，服务器自己留下的那个密钥称为私钥；公钥加密的密文，只能使用私钥来解开，而私钥只有服务器才能知道，不对任何人公开

![非对称加密实现.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adadd728849d4120963e431b71e7ed7c~tplv-k3u1fbpfcp-watermark.image?)

> **过程**：

- 首先浏览器还是发送加密套件列表给服务器。
- 然后服务器会选择一个加密套件，不过和对称加密不同的是，使用非对称加密时服务器上需要有用于浏览器加密的公钥和服务器解密 HTTP 数据的私钥，由于公钥是给浏览器加密使用的，因此服务器会将加密套件和公钥一道发送给浏览器。
- 最后就是浏览器和服务器返回确认消息。

- 浏览器端就有了服务器的公钥，在浏览器端向服务器端发送数据时，就可以使用该公钥来加密数据；即便黑客截获了数据和公钥，他也是无法使用公钥来解密数据


> **存在的问题**：

- 非对称加密的效率太低，严重影响到加解密数据的速度，进而影响到用户打开页面的速度

- 无法保证服务器发送给浏览器的数据安全。服务端采用私钥来加密，客户端能用公钥解开，但是黑客是可以获取公钥的，这样不能保证服务器端数据的安全；


### 对称加密和非对称加密的组合

- 在传输数据阶段依然使用对称加密，但是对称加密的密钥我们采用非对称加密来传输

![混合加密实现.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40e18c54d931461ab1c64902c3ecb597~tplv-k3u1fbpfcp-watermark.image?)

> **过程**：

- 首先浏览器向服务器发送对称加密套件列表、非对称加密套件列表和随机数 client-random；

- 服务器保存随机数 client-random，选择对称加密和非对称加密的套件，然后生成随机数 service-random，向浏览器发送选择的加密套件、service-random 和公钥；

- 浏览器保存公钥，并生成随机数 pre-master，然后利用公钥对 pre-master 加密，并向服务器发送加密后的数据；

- 最后服务器拿出自己的私钥，解密出 pre-master 数据，并返回确认消息。

> **存在的问题**：

- 假如黑客伪造目标服务器站点，黑客就可以自己实现公私钥，服务器如何向浏览器证明 “我就是我” 的问题


### 数字证书

- 引入第三机构CA（Certificate Authority）颁发数字证书Digital Certificate)来证明服务器身份
- 数字证书有两个作用：一个是通过数字证书向浏览器证明服务器的身份，另一个是数字证书里面包含了服务器公钥。

![数字证书.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06e04696db9d4af59fedb05ba2f1f6ad~tplv-k3u1fbpfcp-watermark.image?)

 > **过程**：

相对于组合方式加密俩说，多出来以下两种步骤

- 服务器没有直接返回公钥给浏览器，而是返回了数字证书，而公钥正是包含在数字证书中的；

- 在浏览器端多了一个证书验证的操作，验证了证书之后，才继续后续流程。



## 浏览器验证证书的流程

> 过程：

- 证书的有效期 -> 证书是否被 CA 吊销 -> 证书是否是合法的 CA 机构颁发的。


> 步骤一：证书的有效期

- 证书里面就含有证书的有效期，所以浏览器只需要判断当前时间是否在证书的有效期范围内即可

> 步骤二：验证数字证书是否被吊销了

- 验证
- 第一种方式下载撤销证书列表 [CRL(Certificate Revocation Lists)](https://link.zhihu.com/?target=https%3A//tools.ietf.org/html/rfc5280)

- 第二种方式在线认证[OCSP（Online Certificate Status Protocol ）](https://link.zhihu.com/?target=https%3A//tools.ietf.org/html/rfc2560)浏览器从在线OCSP服务器（也称为OCSP Response Server）请求证书的撤销状态，OCSP Server予以响应


> 步骤三：验证极客时间的数字证书是否是 CA 机构颁发的

 
![浏览器证书认证.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f59a8b35b9f43b2b3b48ded409c9ffc~tplv-k3u1fbpfcp-watermark.image?)

验证证书的流程：
 - 首先，浏览器利用证书的原始信息计算出信息摘要；然后，
 - 利用 CA 的公钥来解密数字证书中的数字签名，解密出来的数据也是信息摘要；
 - 最后，判断这两个信息摘要是否相等就可以了。

 >问题一：浏览器是怎么获取到 CA 公钥的

 1. 服务器直接发送给浏览器，是浏览器直接取到 CA 的公钥
 2. 浏览器还可以通过网络去下载 CA 证书

 > 问题二：证明 CA 机构的合法性？

- 直接在操作系统中内置这些 CA 机构的数字证书，如果证书在内置在操作系统中，就可以直接取出来公钥，`**因为浏览器默认信任操作系统内置的证书为合法证书**`

- 数字证书链：
比如知乎的证书路径


![知乎证书路径.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30655d20364d406f8fdeec0e3219f205~tplv-k3u1fbpfcp-watermark.image?)

 - *.zhihu.com -> GeoTrustCN RAS CA G1 -> DigiCert

 因此浏览器验知乎的证书时，会先验证 *.zhihu.com 的证书，如果合法，再验证中间 CA 的证书，如果中间 CA 也是合法的，那么浏览器会继续验证这个中间 CA 的根证书。

> 那浏览器怎么证明根证书是合法的?

- 简单地判断这个根证书在不在操作系统里面，如果在，那么浏览器就认为这个根证书是合法的；因为如果某个机构想要成为根 CA，并让它的根证书内置到操作系统中，那么这个机构首先要通过 WebTrust 国际安全审计认证


## 参考链接

- [极客时间-浏览器工作原理与实践](https://time.geekbang.org/column/article/180213)
- [证书吊销机制](https://zhuanlan.zhihu.com/p/75475419)
- [加密套件](https://www.openssl.net.cn/docs/215.html)
