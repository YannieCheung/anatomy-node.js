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
当创建了客户端的UPD Socket后，就可以用它的`send()`方法向网络发送消息。与TCP的`write()`方法相比，`send()`方法参数较为复杂，要传递发送的Buffer、Buffer的偏移量、Buffer的长度、目标端口、目标地址、发送完成后的回调。然而它可以随意的发送数据到网络中的服务端。而TCP发送给另一个服务端，就需要重新通过Socket构造一个新的连接。

###事件
除了以上的`listening`和`message`事件，还有以下两个事件：
`close`: 调用close()方法时触发该事件。
`error`:  当异常发生时触发，如果不监听，异常将直接抛出，使进程退出。