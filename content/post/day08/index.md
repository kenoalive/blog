---
title: 前端跨域方案大全
date: 2024-02-14 19:00:00
image: cover.png
categories: ["前端"]
weight: 3 
---
> 前端跨域问题通常发生在浏览器的同源策略限制下，即浏览器只允许同源的脚本相互交互。跨域问题可以通过以下几种方式来解决

### JSONP (JSON with Padding)

JSONP是一种古老的跨域解决方案，它利用了`<script>`标签不受同源策略限制的特性。服务器端将数据包装在一个回调函数中，客户端定义这个回调函数，然后通过动态创建`<script>`标签的方式向服务器端发送请求。

```js
function handleResponse(data) {
  console.log(data);
}

// 动态创建script标签
var script = document.createElement('script');
script.src = 'http://example.com/api?callback=handleResponse';
document.body.appendChild(script);
```

### CORS (Cross-Origin Resource Sharing)

CORS是一种现代的跨域解决方案，它允许服务器决定哪些跨域请求被允许，而不是使用JSONP的"所有请求都允许"策略。服务器端设置适当的CORS头部，客户端发送带有Origin字段的请求，服务器端检查Origin是否在允许列表中，并在响应头中设置Access-Control-Allow-Origin字段。

```js
// 客户端
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://example.com/api', true);
xhr.withCredentials = true; // 表示是否允许发送cookie
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4 && xhr.status === 200) {
    console.log(xhr.responseText);
  }
};
xhr.send();

// 服务器端设置CORS头部
res.setHeader('Access-Control-Allow-Origin', 'http://yourdomain.com');
res.setHeader('Access-Control-Allow-Credentials', 'true');
```

### 代理转发

在开发环境中，可以通过设置代理服务器来解决跨域问题。在webpack-dev-server中，可以通过proxy配置项来设置代理。

```js
module.exports = {
  //...
  devServer: {
    proxy: {
      '/api': {
        target: 'http://example.com',
        changeOrigin: true,
      },
    },
  },
};
```

### WebSockets
WebSockets是一种全双工通信协议，它可以绕过同源策略的限制。服务器端和客户端可以通过WebSocket连接进行通信。

```js
var socket = new WebSocket('ws://example.com/api');
socket.onmessage = function(event) {
  console.log(event.data);
};



```
### postMessage
postMessage是HTML5提供的一种跨文档通信API，它可以用于不同源的窗口、iframe和worker之间的通信。

```js
// 发送消息
window.postMessage('Hello from parent window', 'http://example.com');

// 接收消息
window.addEventListener('message', function(event) {
  if (event.origin !== 'http://example.com') {
    return;
  }
  console.log(event.data);
});
```

### 使用iframe
通过在页面上嵌入一个iframe，并将iframe的源设置为目标服务器，然后通过iframe的contentWindow属性来访问跨域的资源。
```html
<iframe src="http://example.com/api" id="myIframe"></iframe>

<script>
  var iframe = document.getElementById('myIframe');
  iframe.onload = function() {
    var data = iframe.contentWindow.document.body.innerText;
    console.log(data);
  };
</script>

```



