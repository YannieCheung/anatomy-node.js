###Process Events
`process`对象是<a href='./README.md?_k=ezo0xe'>`EventEmitter`</a>类的一个实例
###process.send(message[,sendHandle[,options]][,callback])
massage `<object>` 发送的消息为一个对象

如果程序由IPC创建，那么`process.send()`方法可以用于向父进程发送消息。父进程可以通过子进程对象上的`message`事件来接收消息。

如果程序没有通过IPC创建，那么`process.send()`的值为`undefined`。

_注意:通信时消息会进行序列化和反序列化，接收的洗洗可能与最初的消息不太一样。_