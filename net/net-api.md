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

###new net.Socket([options])
创建一个socket对象。
新创建的socket可以是TCP socket也可以是IPC端点，由[`socket.connect()`]()决定。
`option`可以有以下属性
* fd 用于指定一个存在的socket的文件描述符，TCP客户端使用这个socket与服务端相连接
* readale `<boolean>` 当`fd`被设置时允许socket进行读操作 默认false
* writable `<boolean>` 当`fd`被设置时允许socket进行写操作 默认false
* allowHalfOpen 该属性值被指定为false时，当TCP服务器接收到客户端发送的一个FIN包时将会回发一个FIN包，当该属性被设置true时，当TCP服务器接收到客户端发送的一个FIN包时不会发FIN包，这使得TCP服务器可以继续向客户端发送数据，但是不会接收客户端发送的数据。开发者必须调用`end`方法来关闭socket连接。该属性的默认值为false。

###Event 'close'
事件监听器参数:
* had_error `<boolean>` 如果socket有传输错误，则值为true

一旦socket完全关闭触发该事件。

###Event 'connect'
当socket连接建立成功时触发，具体参见[`net.createConnection()`]()
###Event 'data'
监听器参数
* data `<Buffer>`
当socket接收到数据时触发。参数`data`是一个`Buffer`或`String`。用`socket.setEncoding()`方法设置data的编码。(参见 [Readable Stream]())。
注意如果没有为`Socket`指定一个`data`事件，数据将被丢失。

###Event 'drain'
使用`socket.write()`把数据写入内存，再从内存刷新到系统的缓冲区，接着传输数据。
当写入的缓冲区为空时，触发该事件，可以用来做限制传输的操作。
参见[`socket.write()`]()
###Event 'end'
当连接中的任意一端发送FIN数据时触发该事件
###Event 'error'
当异常发生时，触发该事件
###Event 'lookup'
在解析主机后连接之前触发。不可用于UNIX
###Event 'timeout'
与`setTimeout()`方法对应，用于在一段时间socket不活动了，触发`timeout`事件，在事件回调用户需要手动关闭该连接。
参见[socket.setTimeout()]()
###socket.address()
返回`socket`绑定的地址 如:`{port: 12346, family: 'IPv4', address: '127.0.0.1'}`
###socket.bufferSize
~~**当前准备写入缓冲区的字符数，用户可根据此属性对数据流进行控制。遇到很大或增长很快的`bufferSize `时，用户可用尝试用`pause()` 和 `resume()`来控制字符流。**~~

###socket.destroy([exception])、socket.destroyed
使用这个方法直接关闭连接，要确保在此条socket上已经没有I/O活动了(比如客户端定时发送心跳包，现已超时)，可以在有错误的情况下有必要使用它(解析错误等等)。

如果指定了`exception`，那么当`error`事件被触发时，其侦听器会接受这个`exception`作为参数。

`socket.destroyed`返回连接是否完全关闭

###socket.end([data][,encoding])
该方法封装TCP的半关闭功能，即服务端发送一个FIN包给客户端，关闭客户端到服务端的这条连接通道，但是服务端到客户端的连接还可以继续发送数据，等数据发送完毕再关闭。
```javascript
socket.end(data,encoding);
//等价于
socket.write(data,encoding);
socket.end();
```

###socket.bytesWritten、socket.bytesRead
发送的总字节数，和接收到的总字节数(注意`bytesRead`要在`data`事件的回调中调用才有用，之前还没接收时是0字节)。

###socket.setEncoding([encoding])
* 返回 `<net.Socket>`

设置数据编码，`socket`本身也是个[`Readable Stream`]()

###socket.remotePort、remoteFamily、remoteAddress
返回连接彼端socket的端口、ip版本和地址。

###socket.localPort、localAddress
返回连接此端的端口和地址。

###socket.pause() 、socket.resume()
`pause()`方法使程序停止读取数据，接收到数据，`data`事件也不会触发，`resume()`则重启数据读取功能，触发`data`事件，这有助于节省网络传输的开销。
```javascript
//server.js
...
console.log('客户端与服务器端连接已经建立。');
socket.setEncoding('utf8');
socket.pause();
setTimeout(function(){
    socket.resume();
},5000);
socket.on('data',function(data){
    console.log('已经接收客户端发送的数据: '  + data);
    socket.write('确认数据: ' + data);
    console.log('123');
    socket.end();
});
...
//client.js
...
client.connect(8431,'127.0.0.1',function(){
    console.log('已经连接到服务器端。');
    client.write('你好。');
    console.log('123');
});
...
```

###socket.setKeepAlive([enable][, initialDelay])
* 返回 `<net.Socket>`

是否开启[`keep-alive`](./keepalive)功能，默认不开启，`enable`默认为false。`initialDelay`为从最后一次接收包到发送空包的时间间隙。
这个API主要是给我们一种途径，防止对方的时限过短而断开连接。使用setKeepAlive 会向对方发送 一个空的ACK包，来保持通信。
```javascript
const server = net.createServer(function(socket) { 
    socket.setKeepAlive(true,3000); //如果3s内没有包的交流的话，会发送空包，进行交互
});
```


###socket.setNoDelay([noDelay])
* 返回 `<net.Socket>`

该API主要涉及的是关于网络传输的一个算法--Nagle algorithm. 该算法主要针对的是网络传输时资源节省的算法。 有时候，当你在发送数据包的时候，有时候传输的数据很小，就几个b，老子TCP都比你大不止10倍(通常一个TCP大小是40 byt)。所以为了针对这样的数据包，使用Nagle 可以将小包 缓存起来，等到下一次 打包来的时候，一起发送过去。 but!!! 我们得想想这个算法出来的背景，那时候我们还在使用核桃机，用的是2G网。 能节约1b 是1b 啊，针对那时候 这个算法，是极其有价值的。 but now, 我觉得应该被淘汰了，因为现在4G 已经风靡， 我已经不想再放那张洗脑的2G,3G,4G图了。 大家可以自行脑补。 不过，系统是默认开启的，所以，这样造成的后果是，信息的延迟过大。当今，推荐是关闭该算法，在nodeJS中可以使用setNoDelay(true) 来实现
```javascript
const server = net.createServer(function(socket) { 
    socket.setNoDelay(true);
});
```

###socket.setTimeout(timeout[, callback])
* 返回 `<net.Socket>`
设置`socket`延时`timeout`毫秒再对连接做响应，默认情况下`socket`不会延时。
定时器触发后，`socket`上会触发`timeout`事件，但是连接不会断开。需要用户手动调用`socket.end()`或`socket.destroy()`断开连接。
```javascript
socket.setTimeout(3000,function(){
    console.log('timeout callback...');
});
socket.on('timeout', () => {
    console.log('timeout event callback...');
    socket.end('client disconnect...');
});
//print:
//timeout callback...
//timeout event callback...
```
`callback`将在`timeout`事件触发时执行一次。优先于`timeout`的侦听器执行。

###socket.write(data[, encoding][, callback])
通过socket发送数据。第二个参数指定数据编码，默认utf8。

如果所有数据都被刷新到内核的缓冲区中，该方法返回true。 如果有数据还停留在用户的内存中，返回true。当缓冲区被释放时触发`drain`事件。

`callback`回调会在数据完全写入时调用，可能不会立刻执行。


























































































































































































































































































