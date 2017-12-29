# Events

大多数的Node.js核心API都是围绕一个异步的事件驱动模型而编写的。事件发生器对象(emitter)生成一些特定的事件，然后通过<a href='#tick'>事件循环机制</a>执行回调函数，这些回调函数称为监听器(listener)。这是一个典型的<a href='./shi-jian-fa-5e03-ding-yue-mo-shi.md?_k=hscwr7'>事件发布/订阅模式</a>的应用。

```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
myEmitter.emit('event');
```
1.所有的事件发生器都是`EventEmitter`的实例
2.每个事件发生器都有一个`on()`方法，用于把一个或多个监听器绑定在事件上
3.当事件发生器调用`emit()`方法发生一个事件时，该事件上的监听器将会被调用
4.`emit()`方法是依靠事件循环异步调用的

除了`emit()`可以主动的触发事件外，很多情况下是被动的触发事件，如IO操作、接受web请求、读取一个stream等等。

<a name='tick'></a>
![](/assets/tick流程图.png)