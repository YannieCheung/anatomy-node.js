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
