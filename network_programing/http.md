##HTTP简述
对于构造高性能的网络应用，需要从网络传输层着手，如TCP和UDP都是传输层的协议。

但是对于经典的应用场景，则无需那么复杂，比如HTTP或SMTP等，这些应用层的协议对于普通应用而言绰绰有余。Node提供基本的http和https模块用于对HTTP和HTTPS进行封装，对于其他应用层协议，也能很容易在社区找到实现方案。

在Node中构建HTTP服务及其容易，比如以下官网上实现一个HTTP服务器的经典例子，只是寥寥几行的代码。

```javascript
var http = require('http');
http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type':'text/plain'});
    res.end('Hello World\n');
}).listen(1337, '127.0.0.1');
console.log('Server running at http://127.0.0.1:13337/');
```