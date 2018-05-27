# 整理一些平时比较模糊的知识

## 1、跨域

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