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

异步方法，绑定socket到网卡时，触发`listening`事件，执行`callback`回调。

所有的`listen()`方法都有个[`backlog`](./backlog.md)参数，用于指定那些即将发生的连接的队列的最大长度，默认511，实际情况要参考操作系统本身的最大长度。

注意:
* 所有的 [`net.Socket`]()被设置成`SO_REUSEADDR`。

(一般来说，一个端口释放后会等待两分钟之后才能再被使用，SO_REUSEADDR是让端口释放后立即就可以被再次使用。)

* `server.listen()`方法会被调用多次，随后每次调用将使用提供的参数重新打开服务。

当侦听时可能会发生一个错误，`EADDRINUSE`。当一个工作进程已经占用了该端口(`port`/`path`/`handle`)时会发该错误。一种解决办法是过一段时间重试:
```javascript
server.on('error', (e) => {
  if (e.code === 'EADDRINUSE') {
    console.log('Address in use, retrying...');
    setTimeout(() => {
      server.close();
      server.listen(PORT, HOST);
    }, 1000);
  }
});
```

###server.listen(handle[, backlog][, callback])
一个`handle`已经被绑定到一个端口，或`domain socket`，或`named pipe`上，那么一个服务可以基于这个`handle`监听连接。
_`handle`对象可以是一个`net.Server`，一个`net.Socket`(任何在底层含有`_handle`成员的对象)或一个具有有效文件描述符(`fd`)的对象_

注意:windows上不支持监听一个文件描述符。

###server.listen(options[, callback])
* option 必须。支持以下属性:
  - port
  - host
  - path
  - backlog
  - exclusive ~~集群相关~~
* 返回 `<net.Server>`

如果没有指定`option`，会抛出错误。
如果指定`port`，等价于[`server.listen([port][, hostname][, backlog][, callback])`]()
如果指定`path`，等价于[`server.listen(path[, backlog][, callback])`]()

###server.listen(path[, backlog][, callback])
* path `<String>` 被侦听服务端路径，参见[`Identifying paths for IPC connections`]()
在给定`path`上开启IPC服务侦听。

###server.listen([port][, host][, backlog][, callback])
在给定的端口与主机上开启TCP服务侦听。
如果没有指定`port`或指定为0，操作系统将任意分配一个未使用的端口，在`listening`事件触发后可用`server.address().port`检索该端口。
如果`host`没有指定，则当IPv6可用时，服务器用IPv6地址接收连接，否则将接收IPv4地址的连接。
注意：对于大多数操作系统，侦听IPv6(::)的地址可能会引起`net.Server`也侦听IPv4(0.0.0.0)的地址

###server.listening
一个布尔值，指示服务器是否正在侦听连接。

###server.maxConnections
设置服务端最大连接数量，当到达该值时，服务器将拒绝连接。
如果socket已经被发送给子进程(`child_process.fork()`)，那么最好是不要用这个选项

###server.unref()
* 返回 `<net.Server>`

如果`server`是在事件系统中唯一激活的服务端，那么调用`unref()`方法将退出程序。如果`server`已经执行过`unref()`方法，再次调用`unref()`将不会有效果

###server.ref()
* 返回 `<net.Server>`

与`unref()`相反，在被`unref()`的`server`上调用`ref()`，那么程序会加如事件系统。连续的调用`ref()`也不会有效果。

##Class: net.Socket
`net.Socket`是TCP socket或者IPC的抽象。它也是一个[`duplex stream`]()，所以它是可读可写的，并且它还是个[`EventEmitter`]()。

`net.Socket`可以被用户创建并用来直接和服务端交互，例如，[`net.createConnection()`]()返回`net.Socket`的一个对象，用户用它与服务端交互。它也可以由Node.js自己产生，在收到连接时可以传递给用户。例如，[`connection`]()事件被触发时，`net.Socket`会传递给其侦听器，以便用它与客户端交互。














































































































































