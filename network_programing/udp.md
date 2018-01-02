#UDP简述
在TCP中客户端如果要与不同的TCP服务通信，需要创建不同的Socket来完成连接。而在UDP中，一个Socket可以与多个UDP服务通信。但是UDP的信息传输是不可靠的，在网络差的情况下存在严重的丢包问题，但由于它无须重新连接，资源消耗低，处理快速且灵活，因此经常用于那种偶尔丢一两个数据包也不会产生重大影响的场景，如音频、视频等。DNS服务也是基于它实现的。

###创建服务端
以下为一个完整服务端示例
```javascript
const dgram = require('dgram');

const server = dgram.createSocket('udp4');

server.on("message", function(msg, rinfo){
    console.log("server got: " + msg + " from " + 
        rinfo.address + ":" + rinfo.port);
});

server.on("listening", function(){
    var address = server.address();
    console.log("server listening " + address.address + ":" + address.port);
});

server.bind(41234);
```
以上代码，首先创建UPD的Socket，Socket一旦创建，即可作为客户端发送数据，也可以作为服务端接受数据。之后在为创建的服务端上的两个事件`message`和`listening`注册handler。事件`message`在Socket侦听网卡端口后，接受到消息时触发，触发时会向handler传递两个参数，携带数据的Buffer对象和远程地址信息。事件`listening`在Socket绑定到网卡端口可以接受消息时触发。
###创建客户端
以下为一个客户端示例
```javascript
const dgram = require('dgram');

var message = new Buffer("网络编程 UDP");
var client = dgram.createSocket("udp4");
client.send(message, 0, message.length, 41234, "localhost", function(err, bytes){
    client.close();
});
```
当创建了客户端的UPD Socket后，就可以用它的`send()`方法向网络发送消息。