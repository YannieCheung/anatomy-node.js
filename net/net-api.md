#Net
稳定性:2

为了创建基于`stream`的TCP或者[`IPC`]()服务，`net`模块提供一套异步网络API。

使用:
```javascript
const net = require('net');
```
##IPC Support
`net`模块支持[IPC](./../process/IPC principle.md)(进程间通信)，在windows上是依靠`named pipes`实现，在*nix上是依靠`domain sockets`实现。

###为IPC连接标识路径
[]()，[]()，[]()，和[]()这些方法使用`path`参数识别IPC端口。

##Class:net.Server
这个类可以用于创建一个TCP或IPC服务

###new net.Server([option][,connectionListener])
* 返回 `<net.Server>`

参见 [`net.createServer([options][, connectionListener])`]()。
`net.Server`是一个`EventEmitter`的实例，有如下事件:

###Event:'close'
当服务器关闭时触发，在调用server.close()后，服务器将停止接受新的socket连接，但保持当前存在的连接，等所有连接断开后，会触发该事件。
###Event:'connection'
当客户端的socket连接到服务端时触发，简洁的写法是通过net.createServer()的最后一个参数传递那个回调函数。
###Event:'error'
当服务器发生异常时触发，如监听一个已经在使用中的端口，会发生异常，如果不侦听error事件，整个服务器将抛出异常。

还有不同于[`net.Socket`]()的`close`事件，`close`不会被自动触发，除非手动调用[` server.close()`]()方法。参见[`server.listen()`]()中的例子。
###Event:'listening'
在调用[`server.listen()`]()方法绑定端口或者`Domain Socket`后触发，简介的写法为[`server.listen(port,listeningListener)`]()，通过第二个参数传入。


###sever.address()
返回服务端的监听端口，IP版本和IP地址，如`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`。
```javascript
var net = require('net');
const server = net.createServer((socket) => {
    socket.end('goodbye\n');
}).on('error', (err) => {
    // handle errors here
    throw err;
});

// 分配任意一个未使用的端口
server.listen(() => {
    console.log('opened server on', server.address());
});
```
当`listening`事件触发时才调用`server.address()`。


###server.close([callback])
* 返回 `<net.Server>`

阻止服务器接收新的连接并维持已存在的连接。该方法是异步的，当所有连接结束后服务器最终将关闭并触发`close`事件，参数`callback`函数将在`close`触发时调用。
_如果服务器还没打开就关闭，那么该回调将直接被调用。_

###server.getConnections(callback)
* 返回 `<net.Server>`
异步方法，获取服务端上的并发连接数。当socket被发送给子进程时也可以使用
`callback`有两个参数`err`和`count`，`count`即连接数。

###server.listen()
为服务器侦听连接。服务端可以是TCP或IPC，取决于侦听的内容。
可以是以下签名。
* [`server.listen(handle[, backlog][, callback])`]()
* [`server.listen(options[, callback])`]()
* [`server.listen(path[, backlog][, callback])`]() 针对IPC
* [`server.listen([port][, host][, backlog][, callback])`]() 针对TCP










































































