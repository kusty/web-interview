# 整理一些平时比较模糊的知识

## 1、跨域

### 什么是跨域

> 如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的源。浏览器的同源策略，限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。不同源直接的交互就叫做跨域。

### 跨域的几种方法

- jsonp

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

jsonp的缺点为只支持get方式