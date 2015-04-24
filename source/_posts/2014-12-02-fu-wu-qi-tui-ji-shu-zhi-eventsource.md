---
layout: post
title: "服务器推技术之EventSource"
tags:
    - 服务器推
    - polling
    - long polling
    - WebSocket
    - EventSource
description: ""
---

最近在看服务器推技术时，发现了一项新的技术：__EventSource__。

w3c上有个草案描述了它：[Server-Send Events](http://dev.w3.org/html5/eventsource/)。

回想一下我们之前用到的服务器推技术：

* `polling`：客户端不断轮询。
* `long polling`：服务端在数据未就绪时，挂起请求。

这两种技术都存在一定的局限，我们需要服务器主动推送数据。

于是有了`WebSocket`，但它是双向的（服务端<——>客户端）。

有的时候，我们并不需要从客户端发送消息，我们只需要从服务端推送消息，`EventSource`应运而生。

`EventSource`是单向的（服务端——>客户端），它直接使用http协议来传输数据（与`WebSocket`不同，`EventSource`不需要专门的协议）。

<!-- more -->

客户端, client.html：

```javascript
if (window.EventSource) {
    var source = new EventSource('/stream');

    source.addEventListener('message', function(e) {
        console.log(e.data);
    }, false);
} 
```

服务端, server.js：

```javascript
var http = require('http');
var fs = require('fs');

http.createServer(function(req, res) {
  if (req.headers.accept && req.headers.accept == 'text/event-stream') {
    if (req.url == '/stream') {
      sendSSE(req, res);
    } 
    else {
      res.writeHead(404);
      res.end();
    }
  } 
  else {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.write(fs.readFileSync(__dirname + '/client.html'));
    res.end();
  }
}).listen(8000);

function sendSSE(req, res) {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });

  var id = (new Date()).toLocaleTimeString();

  setInterval(function() {
    constructSSE(res, id, (new Date()).toLocaleTimeString());
  }, 5000);

  constructSSE(res, id, (new Date()).toLocaleTimeString());
}

function constructSSE(res, id, data) {
  res.write('id: ' + id + '\n');
  res.write("data: " + data + '\n\n');
}
```

`EventSource`的`content-type`是`text/event-stream`，数据传输格式：

    data: xxxxxxx\n\n

多行的数据：

    data: xxxxxxxx\n
    data: xxxxxxxx\n\n

最后一行使用两个`\n`。

我们可以使用`EventSource`传输json格式数据：

    data: {\n
    data: "name": "hushicai"\n
    data: }\n\n

客户端接收到数据后，可以调用`JSON.parse`解析出json数据。

## 参考文章

* http://www.html5rocks.com/en/tutorials/eventsource/basics/
