###Process Events
`process`对象是<a href='./README.md?_k=ezo0xe'>`EventEmitter`</a>类的一个实例
####事件:message
如果Node.js进程是由IPC创建的(查看[Child Process](./process/child-process.md)和[Cluster]()文档)，在父进程中使用子进程对象的[`childprocess.send()`]()方法发送消息时，在子进程中就会触发`message`事件。

侦听器上绑定以下参数:
* message `<Object>` 一个JSON对象或者原生类型的值
* sendHandle `<Handle object>` 一个[`net.Socket`]()或[`net.Server`]()对象或为undifined

_注意:通信时消息会进行序列化和反序列化，接收的消息可能与最初的消息不太一样。_


###process.send(message[,sendHandle[,options]][,callback])
massage `<object>` 发送的消息为一个对象

如果程序由IPC创建，那么`process.send()`方法可以用于向父进程发送消息。父进程可以通过子进程对象上的`message`事件来接收消息。

如果程序没有通过IPC创建，那么`process.send()`的值为`undefined`。

_注意:通信时消息会进行序列化和反序列化，接收的消息可能与最初的消息不太一样。_