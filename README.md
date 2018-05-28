# 整理一些知识点

## 跨域

### 什么是跨域

> 如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的源。浏览器的同源策略，限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。不同源直接的交互就叫做跨域。

### 跨域的几种方法

#### jsonp

在html中，script标签没有被同源策略限制，所以我们可以通过动态创建script来实现跨域。具体实现代码如下：

```js
<script>
    var script = document.createElement('script');
    script.type = 'text/javascript';
    //在src中指定需要的参数和回调处理函数名称
    script.src = 'http://www.domain2.com/a?params=params&callback=onBack';
    document.head.appendChild(script);

    // 回调处理函数
    function onBack(data) {
        //处理返回数据
    }
    //返回为onBack(data),数据一旦返回会立即执行onBack函数
 </script>
```

> jsonp的缺点为只支持get方式

#### document.domain

此方案仅限主域相同，子域不同的跨域应用场景。

实现原理：两个页面都通过js强制设置document.domain为基础主域，就实现了同域。一般在iframe里面使用。

#### postMessage跨域

postMessage是HTML5 XMLHttpRequest Level 2中的API，且是为数不多可以跨域操作的window属性之一，它可用于解决以下方面的问题：

a.） 页面和其打开的新窗口的数据传递

b.） 多窗口之间消息传递

c.） 页面与嵌套的iframe消息传递

d.） 上面三个场景的跨域数据传递

用法：postMessage(data,origin)方法接受两个参数

data： html5规范支持任意基本类型或可复制的对象，但部分浏览器只支持字符串，所以传参时最好用JSON.stringify()序列化。

origin： 协议+主机+端口号，也可以设置为"*"，表示可以传递给任意窗口，如果要指定和当前窗口同源的话设置为"/"。

#### 跨域资源共享（CORS）

普通跨域请求：只服务端设置Access-Control-Allow-Origin即可，前端无须设置，若要带cookie请求：前后端都需要设置。需注意的是：由于同源策略的限制，所读取的cookie为跨域请求接口所在域的cookie，而非当前页。

#### nginx配置跨域

- 在nginx配置中加入如下代码

```nginx
location / {
  add_header Access-Control-Allow-Origin *;
}
```

- 也可以利用反向代理接口跨域

通过nginx配置一个代理服务器（域名与domain1相同，端口不同）做跳板机，反向代理访问domain2接口，并且可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域登录。

## HTTP

### 一些常见状态码

```c
100  Continue   继续，一般在发送post请求时，已发送了http header之后服务端将返回此信息，表示确认，之后发送具体参数信息
200  OK         正常返回信息
201  Created    请求成功并且服务器创建了新的资源
202  Accepted   服务器已接受请求，但尚未处理
301  Moved Permanently  请求的网页已永久移动到新位置。
302  Found       临时性重定向。
303  See Other   临时性重定向，且总是使用 GET 请求新的 URI。
304  Not Modified 自从上次请求后，请求的网页未修改过。

400 Bad Request  服务器无法理解请求的格式，客户端不应当尝试再次使用相同的内容发起请求。
401 Unauthorized 请求未授权。
403 Forbidden   禁止访问。
404 Not Found   找不到如何与 URI 相匹配的资源。

500 Internal Server Error  最常见的服务器端错误。
503 Service Unavailable 服务器端暂时无法处理请求（可能是过载或维护）
```

### TCP的三次握手和四次挥手

tcp建立连接的时候需要有3次握手

- 第一次握手：建立连接。客户端发送联机请求报文段(SYN)，进入SYN_SEND状态，等待服务器的确认；

- 第二次握手：服务器收到联机请求报文(SYN)。对报文进行确认，生成确认报文(ACK)，并连同联机请求报文一同发给客户端(SYN+ACK)，进入SYN_RECV；

- 第三次握手：客户端收到服务器的SYN+ACK报文，向服务器发送ACK报文段，完成以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。

tcp断开连接的时候需要有4次挥手，由于TCP连接是全双工的，一个TCP连接存在双向的读写通道，因此每个方向都必须单独进行关闭。这原则是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向的连接。收到一个 FIN只意味着这一方向上没有数据流动，一个TCP连接在收到一个FIN后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。

- 第一次挥手：主动关闭方发送一个FIN，用来关闭主动方到被动关闭方的数据传送，也就是主动关闭方告诉被动关闭方：我已经不 会再给你发数据了(当然，在fin包之前发送出去的数据，如果没有收到对应的ack确认报文，主动关闭方依然会重发这些数据)，但是，此时主动关闭方还可 以接受数据；

- 第二次挥手：被动关闭方收到FIN包后，发送一个ACK给对方，确认序号为收到序号+1(与SYN相同，一个FIN占用一个序号)；

- 第三次挥手：被动关闭方发送一个FIN，用来关闭被动关闭方到主动关闭方的数据传送，也就是告诉主动关闭方，我的数据也发送完了，不会再给你发数据了；

- 第四次挥手：主动关闭方收到FIN后，发送一个ACK给被动关闭方，确认序号为收到序号+1，至此，完成四次挥手。

### HTTP和HTTPS

- HTTP是属于应用层的协议，它是基于TCP/IP的，所以它只是规定一些要传输的内容，以及头部信息，然后通过TCP协议进行传输，依靠IP协议进行寻址，客户端发出请求，服务端进行响应，就是这么简单。在整个过程中，没有任何加密的东西，所以它是不安全的，中间人可以进行拦截，获取传输和响应的数据，造成数据泄露。

- HTTPS就是使用SSL/TLS协议进行加密传输，让客户端拿到服务器的公钥，然后客户端随机生成一个对称加密的秘钥，使用公钥加密，传输给服务端，后续的所有信息都通过该对称秘钥进行加密解密，完成整个HTTPS的流程。

## nginx
