####subprocess.send(message[, sendHandle[, options]][, callback])
当父进程和子进程之间建立一个IPC通道时(例如使用[`child_process.fork`]()方法时)，`subprocess.send()`方法可以用于向子进程发送消息。当子进程为一个Node.js实例时，消息可以通过`process.on('message')`事件接受。

_注意:通信时消息会进行序列化和反序列化，接收的消息可能与最初的消息不太一样。_

例如，在一个父进程脚本中有:
```javascript
const cp = require('child_process');
const n = cp.fork(`${__dirname}/sub.js`);

n.on('message', (m) => {
  console.log('PARENT got message:', m);
});

// Causes the child to print: CHILD got message: { hello: 'world' }
n.send({ hello: 'world' });
```