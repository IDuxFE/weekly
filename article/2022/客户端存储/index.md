# 客户端存储

## 一、背景
现在web浏览器提供了很多在客户端存放数据的方法，只要用户允许，就可以在需要的时候重新获得。这种长时间存储数据的方法，在很多场景都有使用，比如保存用户个性化网站配置，保存已经生成的文档，使得用户在离线状态下可以访问等。

## 二、分类
客户端存储可以分为四大类：cookie，Web Storage，IndexedDB和Cache。

![](https://files.mdnice.com/user/22004/865c1632-917d-402d-a1a4-36045f0b27b9.png)

**cookie**

cookies是网络上最早最常用的客户端存储形式。从早期的网络时代开始，网站就使用cookies来存储数据，用来提供个性化的用户体验。

由于cookie本身存在各种安全问题，并且无法存储复杂数据，而且随着更好用的存储数据方法出现，cookie逐渐被淘汰。但是cookie仍存在优势，它得到了非常旧的浏览器的支持，如果项目中需要支持过时的浏览器，比如IE8，那么它仍然有用。

**Web Storage**

Web Storage定义了简单的键/值存储，用于存储和检索简单数据。有两种API：sessionStorage和localStorage。

Web存储最主要的缺点就是它的阻塞式同步API和在某些浏览器中有限的存储空间。但是其在客户端浏览器上有很好的支持，它是客户端持久化的首选，简单朴素是它的优势。

**IndexedDB**

IndexDB提供了一个完整的数据库系统用于存储复杂的数据类型。其种类不限于字符串或者数字等这种简单的数据类型，可以是音频或者视频文件等。

IndexDB的API比Web Storage复杂，而且浏览器支持也不友好。但是IndexDB是非堵塞的，并且存储空间比Web存储大很多。

**Cache**

Cache是另一种客户端存储机制，为了存储特定HTTP请求的响应文件设计的，与Service Worker组合使用。主要应用场景是存储离线文件，这样网站可以在没有网络连接情况下使用。

这是一个实验中的功能，目前只有一些现代浏览器支持新的Cache API，某些浏览器尚在开发中。由于该功能对应的标准文档可能被重新修订，所以在未来版本的浏览器中该功能的语法和行为可能随之改变。所以不会在本文中讨论它。

打开谷歌浏览器，点击开发者工具，点击Application，查看浏览器缓存如下：

![](https://files.mdnice.com/user/22004/2a8ee9c1-135d-449c-99b7-6f60c525bcef.png)

下面是对以上API的详细介绍。

## 三、cookie

### 3.1 cookie是什么

HTTP Cookie，通常直接叫做Cookie，是服务器发送到用户浏览器并保存在本地的数据，会在浏览器下次向同一服务器再次发起请求时被携带并发送到服务器上。

### 3.2 cookie用途

Cookie主要用于以下三个方面：

会话状态管理（如用户登录状态等）

个性化设置（如用户自定义设置等）

浏览器行为跟踪（如跟踪分析用户行为等）

### 3.3 HTTP创建cookie

服务器对任意HTTP请求发送Set-Cookie HTTP头作为响应的一部分，其中包含会话信息。例如，这种服务器响应的头可能如下：

    HTTP/1.1 200 OK
    Content-Type: text/html
    Set-Cookie: name=Value
    Other-Header: Other-header-value

这个HTTP响应设置以name为名称，以value为值的一个cookie。一个简单的Cookie如下：

    Set-Cookie: <cookie名>=<cookie值>

浏览器会存储这样的会话信息，并在这之后，通过为每个请求添加Cookie HTTP头将信息发送回服务器，如下所示：

    GET /index.html HTTP/1.1
    Cookie: name=Value
    Other-Header: Other-header-value

发送回服务器的额外信息可以用于唯一验证客户来自于发送的哪个请求。

进入掘金页面，查看到此过程示例：

![](https://files.mdnice.com/user/22004/267acb3b-b575-445e-899c-d74f54de1e6d.png)

![](https://files.mdnice.com/user/22004/f038bf10-d94a-4666-82c7-c9f4657e0285.png)

### 3.4 cookie属性

cookie由名称、值、域、路径、失效时间、安全标志等信息构成，可以通过Set-Cookie添加属性，按需指定。

| 属性      | 简介 | 描述 |
| ----------- | ----------- | ----------- |
| name      | 名称       | 以 __Secure- 为前缀的 cookie，必须与 secure 属性一同设置，同时必须应用于安全页面；以 __Host- 为前缀的 cookie，必须与 secure 属性一同设置，必须应用于安全页面，必须不能设置 domain 属性，同时 path 属性的值必须为“/” |
| value   | 值        |  |
| Expires   | 失效时间        | cookie 的最长有效时间 |
| Max-Age   | 失效时间       | 在 cookie 失效之前需要经过的秒数 。假如二者 （指 Expires 和Max-Age） 均存在，那么 Max-Age 优先级更高 |
| Domain   | 域        | 指定 cookie 可以送达的主机名 |
| Path   | 路径    | 指定一个 URL 路径，这个路径必须出现在要请求的资源的路径中才可以发送 Cookie 首部  |
| Secure   | 安全标志        | 一个带有安全属性的 cookie 只有在请求使用SSL和HTTPS协议的时候才会被发送到服务器 |
| HttpOnly   | 安全标志        | 设置了 HttpOnly 属性的 cookie 不能使用 JavaScript 经由  Document.cookie 属性、XMLHttpRequest 和  Request APIs 进行访问，以防范跨站脚本攻击（XSS (en-US)） |
| SameSite   | 安全标志       | 允许服务器设定一则 cookie 不随着跨域请求一起发送，这样可以在一定程度上防范跨站请求伪造攻击（CSRF）  |

### 3.5 JavaScript 访问 Cookie

通过 Document.cookie 属性可创建新的 Cookie，也可通过该属性访问非HttpOnly标记的Cookie。
通过 JavaScript 创建的 Cookie 不能包含 HttpOnly 标志。


    document.cookie = "yummy_cookie=choco";
    document.cookie = "tasty_cookie=strawberry";
    console.log(document.cookie);
    // logs "yummy_cookie=choco; tasty_cookie=strawberry"

控制台设置后，即可看新增设置的cookie：

![](https://files.mdnice.com/user/22004/44ab6332-0d1f-48b0-8f3b-b01c3e91b299.png)

## 四、Web Storage

### 4.1 Web Storage是什么

Web Storage的目的是克服由cookie带来的一些限制，当数据需要被严格控制在客户端上时，无需持续地将数据发回服务器。Web Storage的两个主要目的是：
+ 提供一种在cookie之外存储会话数据的途径
+ 提供一种存储大量可以跨会话存在的数据的机制。

### 4.2 Web Storage分类

Web Storage包含了两种对象的定义： sessionStorage和localStorage。两者区别在于存储的有效期和作用域不同。

|       | sessionStorage | localStorage |
| ----------- | ----------- | ----------- |
| 存储有效期      | 窗口或者标签页被关闭，数据将被删除       | 永久性的，除非刻意删除，否则永不过期 |
| 作用域   | 文档源级别，非同源的文档间不共享；还被限定在窗口中，即同源的文档渲染在不同的浏览器标签页中，无法共享        | 文档源级别，同源的文档间共享，非同源的文档间不共享 |

### 4.3 基本语法
本文中使用localStorage方法举例，因为它通常更有用。

**存储**

Storage.setItem() 方法存储一个数据项——它接受两个参数：数据项的名字及其值。

    localStorage.setItem('name','Alice');

输入到JavaScript控制台，可以看到效果：

![](https://files.mdnice.com/user/22004/b33bf6b1-438e-48a2-92f3-de2d7bc4cfd8.png)

**获取**

Storage.getItem() 方法获取一个数据项—它接受一个参数：检索的数据项的名称，返回数据项的值。

    localStorage.getItem('name');

将这些代码输入到JavaScript控制台，看到效果：

![](https://files.mdnice.com/user/22004/3c793ca1-8b7d-4499-b2e1-88978ee275f0.png)

**删除**

Storage.removeItem() 方法删除一个数据项-它接受一个参数：将要删除的数据项的名称，并从 web storage 中删除该数据项。

    localStorage.removeItem('name');
    localStorage.getItem('name');

将这些代码输入到JavaScript控制台，看到效果：

![](https://files.mdnice.com/user/22004/8dee2d06-b89d-4b26-adb7-8e7af969654a.png)

![](https://files.mdnice.com/user/22004/40f773fa-d46c-4633-a327-d45c19f07d52.png)

### 4.4 限制
与其他客户端数据存储方案类似，Web Storage同样也有限制。这些限制因浏览而异。一般来说，对存储空间大小的限制都是以每个来源为单位的，也就是每个来源都有固定大小的空间用于保存自己的数据。不同的浏览器对localStorage和sessionStorage有不同的容量限制。

## 五、IndexedDB

### 5.1 IndexedDB是什么

Indexed Database API，简称IndexedDB，是在浏览器中保存结构化数据的一种数据库。IndexedDB是为了替代目前已经废弃的Web SQL Database API而出现的。IndexedDB的思想是创建一套API，方便保存和读取JavaScript对象，同时还支持查询和搜索。


### 5.2 关键概念和设计思想

IndexedDB设计的操作完全是异步进行的。因此，大多数操作会以请求方式进行，但这些操作会在后期执行，如果成功则返回结果，如果失败则返回错误。差不多每一次IndexedDB操作，都需要注册onerror或onsuccess事件处理程序，以确保适当地处理结果。

### 5.3 使用方法

#### 5.3.1 数据库初始设置

使用IndexedDB的第一步是打开它，即把要打开的数据库名传给indexDB.open()。如果传入的数据库已经存在，就会发送一个打开它的请求；如果传入的数据库还不存在，就会发送一个创建并打开它的请求。总之，调用indexDB.open()会返回一个IDBRequest对象，在这个对象上可以添加onerror或onsuccess事件处理。

    var request, database;
    request = indexDB.open("databaseName");
    request.onerror =  function() {
        console.log('Database failed to open');
    }
    request.onsuccess =  function() {
        console.log('Database opened successfully');
    }

#### 5.3.2 添加数据到数据库

在建立了与数据库的连接之后，下一步就是使用对象存储空间。在创建对象存储空间时，必须指定一个全局唯一的键。如下为使用对象中id的值为全局唯一的键。

    db = request.result;
    var store = db.createObjectStore('uniqueKey', { keyPath: "id" }) 

接下来可以使用add()或put()方法来向其中添加数据。这两个方法都接受一个参数，即要保存的对象，然后这个对象就会被保存在存储空间内。二者区别在于，在空间中已经包含键值相同的对象时，add()会返回错误，put()会重写原对象。

    var user = {
        username: '007',
        password: 'foo'
    }
    store.add(user)


#### 5.3.3 显示数据

创建对象存储空间完成后，接下来的所有操作都是通过事务来完成的。在数据库对象上调用transaction()方法可以创建事务。只要想读取或修改数据，都要通过事务来组织所有操作。

+ 如果没有参数，是通过事务来读取数据库中保存的对象。

    var transaction = db.transaction();

+ 如果只有一个参数，类型可以是字符串或者字符串数组，则为访问一个或者多个对象存储空间：

    var transaction = db.transaction(“user”);

    var transaction = db.transaction(["user", "anotherUser"]);

+ 如果有两个参数，第二个参表示访问模式：readonly， readwrite，readwriteflush (非标准, 只适用Firefox)

    var transaction = db.transaction('user', "readwrite");

取得事务的索引后，使用objectStore()方法并传入存储空间名字，就可以访问特定的存储空间。然后使用add()或put()方法，使用get()取得值，使用delete()删除对象，使用clear()删除所有对象。

### 5.4 限制

首先，IndexedDB数据库只能由同源页面操作，因此不能跨域共享信息。

其次，每个来源的数据库占用的磁盘空间也是有限制。如果超过了原有配额，将会请求用户的许可。

## 六、参考

1. [客户端存储](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Client-side_web_APIs/Client-side_storage)
2. 《JavaScript高级程序设计（第3版）》
