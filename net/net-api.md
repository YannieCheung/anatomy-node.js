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

###`Event`:'close'