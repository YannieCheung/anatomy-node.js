##Net
`net`模块提供一组可以创建基于流的TCP或者IPC服务端和客户端的异步网络API。

##Class:net.Server
这个类用于创建TCP或者IPC服务端。

####new net.Server([options][,connectionListener])
参见 [`net.createServer([options][,connectionListener])`](#crdserv)











####<span id='e_conn'>事件:connection</span>
    - [`<net.Socket>`](#socket) 连接对象
    
当一个新的连接被使用时触发，`socket`是`net.Socket`的一个实例

####<span id='e_listening'>事件:listening</span>
在调用[server.listen()](#listen)后，服务器开始响应时触发。

####<span id='listen'>server.listen()</span>
为一个连接开启服务端监听，一个`net.Server`到底是TCP还是IPC的服务，取决于它监听的是什么。

这个方法是异步调用的。当服务器开始监听时，[`listening`](#e_listening)事件被触发。该方法的最后一个参数是注册在[`listening`](#e_listening)事件上的监听器。

所有形式的`listen()`方法都可以设置一个`backlog`参数，该参数指定连接等待队列的最大长度，实际的长度取决于操作系统的设置，比如Linux上的`tcp_max_syn_backlog`和`somaxconn`。默认值为511。






##<span id='socket'>net.Socket</span>























####<span id='#crdserv'>net.createServer([options][,connectionListener])</span>
* options
    - allowHalfOpen
    - pauseOnConnect
* connectionListener `<Function>` 为[`connection`](#e_conn)事件注册一个监听器
* 返回: `<net.Server>`

下面是一个TCP服务端的例子，在8124端口监听连接。
```javascript
const net = require('net');
const server = net.createServer((c) => {
    console.log('客户端已连接');
    c.on('end', () => {
        console.log('客户端丢失连接');
    };
    c.write('hello\r\n');
    c.pipe(c);
};
server.on('error', (err) => {
    throw err;
});
server.listen(8124, () => {
    console.log('服务端响应');
});
```








