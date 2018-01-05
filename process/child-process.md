####subprocess.send(message[, sendHandle[, options]][, callback])
当父进程和子进程之间建立一个IPC通道时(例如使用[`child_process.fork`]()方法时)，父进程可以通过`subprocess.send()`方法向子进程发送消息。此时，在子进程中，触发`process.on('message')`事件，接受父进程发送的消息。

_注意:通信时消息会进行序列化和反序列化，接收的消息可能与最初的消息不太一样。_

例如，在一个父进程脚本中有:
```javascript
const cp = require('child_process');
const n = cp.fork(`${__dirname}/sub.js`);

n.on('message', (m) => {
  console.log('PARENT got message:', m);
});

// 使子进程打印: CHILD got message: { hello: 'world' }
n.send({ hello: 'world' });
```
而在子进程脚本中有:
```javascript
process.on('message', (m) => {
  console.log('CHILD got message:', m);
});

// 使父进程打印: PARENT got message: { foo: 'bar', baz: null }
process.send({ foo: 'bar', baz: NaN });
```
这里有个特殊情况，当父进程发送的消息为`cmd`属性，值前缀为`NODE_`时，子进程不会触发`process.on('message')`事件，因为这是留给Node.js内核用的。但是，这种类型的消息可以触发留作Node.js内部使用的`process.on('internalMessage')`事件。因此，在程序中应该避免使用此类消息或者在子进程中监听`internalMessage`事件。

关于第二个参数，详见[`句柄传递`]()
参数`sendHandles`可以向子进程传递一个TCP-server/socket对象。子进程[`process.on('message')`]()事件回调函数的第二个参数将会接收到这个对象。_而从socket中接收到的任何数据都不会发送给子进程。_