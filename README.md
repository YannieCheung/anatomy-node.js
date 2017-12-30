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
2.每个事件发生器都有一个`on()`方法，用于把一个或多个监听器绑定在事件上(监听器队列，后来的插入队列尾部)
3.当事件发生器调用`emit()`方法发生一个事件时，该事件上的监听器将会被调用
4.`emit()`方法是依靠事件循环异步调用的

除了`emit()`可以主动的触发事件外，很多情况下是被动的触发事件，如IO操作、接受web请求、读取一个stream等等。

<a name='tick'></a>
![](/assets/tick流程图.png)

###给监听器传递参数
`EventEmitter`通过`emit()`触发事件时，可以向监听器中传递一组参数，监听器在执行时，其中的this指向`EventEmitter`实例。但是使用ES6的函数表达式定义监听器时，this不会指向`EventEmitter`实例

```javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', function(a, b) {
  console.log(a, b, this);
  // Prints:
  //   a b MyEmitter {
  //     domain: null,
  //     _events: { event: [Function] },
  //     _eventsCount: 1,
  //     _maxListeners: undefined }
});
myEmitter.emit('event', 'a', 'b');
```
```javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  console.log(a, b, this);
  // Prints: a b {}
});
myEmitter.emit('event', 'a', 'b');
```
###只执行一次监听器的绑定方式
使用`on()`方法绑定的事件，在每次调用`emit()`触发事件时，监听器都会执行。
```javascript
const myEmitter = new MyEmitter();
let m = 0;
myEmitter.on('event', () => {
  console.log(++m);
});
myEmitter.emit('event');
// Prints: 1
myEmitter.emit('event');
// Prints: 2
```
如果使用`once()`方法绑定事件，监听器只会在首次触发事件时执行一次
```javascript
const myEmitter = new MyEmitter();
let m = 0;
myEmitter.once('event', () => {
  console.log(++m);
});
myEmitter.emit('event');
// Prints: 1
myEmitter.emit('event');
// Ignored
```
`once()`可以巧妙的应用在<a href='./li-yong-shi-jian-dui-lie-jie-jue-xue-beng-wen-ti.md?_k=nsvlxp'>解决雪崩的问题</a>上

###监听器队列的同步与异步
`EventListener`同步地按照监听器注册的顺序去一个个调用它们，这样避免了资源的竞争或逻辑上的错误，还可以确保事件有正确的序列。而有时候，这种同步的监听器调用可以使用`setImmediate()`或者`process.nextTick()`切换成异步模式，关于这两个方法，
可以参见<a href="./asyn_no_io.md?_k=tgbf04">非I/O的异步API</a>
```javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  setImmediate(() => {
    console.log('this happens asynchronously');
  });
});
myEmitter.emit('event', 'a', 'b');
```

###事件newListener和removeListener
`EventEmitter`的实例在一个监听器被添加进它的监听器队列时会触发`newListener`事件。
在一个监听器被移除的时候会触发`removeListener`事件。
而`newListener`自己的监听器会接收两个参数:event(被添加的监听器所对应的事件名称)和listener(被添加的监听器)
```javascript
const myEmitter = new MyEmitter();
// Only do this once so we don't loop forever
myEmitter.once('newListener', (event, listener) => {
  if (event === 'event') {
    // Insert a new listener in front
    myEmitter.on('event', () => {
      console.log('B');
    });
  }
});
myEmitter.on('event', () => {
  console.log('A');
});
myEmitter.emit('event');
// Prints:
//   B
//   A
```
注意:如果在`newListener`的监听器中，注册了一个在`newListener`之外注册的同名事件，那么这个内部的监听器会插入到外部监听器的前面。如上面的例子会先打印内部监听器执行结果。

###方法emitter.listeners(eventName)和emitter.listenerCount(eventName)
`emitter.listenerCount()`返回指定事件上注册的监听器数量
`emitter.listeners()`返回指定事件上注册的监听器数组(回调函数列表)
```javascript
server.on('connection', (stream) => {
  console.log('someone connected!');
});
console.log(util.inspect(server.listeners('connection')));
// Prints: [ [Function] ]
````

###emitter.prependListener(eventName, listener)/prependOnceListener(eventName, listener)
`emitter.on()`是把监听器插入到注册队列尾部，此方法把监听器插入到队列首部
`prependOnceListener()`则是与`once()`方法相对应，只执行一次
```javscript
server.prependListener('connection', (stream) => {
  console.log('someone connected!');
});
```
特别注意，事件一旦被`emit()`触发了，那么注册的所有监听器将会被调用，意思就是说这两个方法不会删除正在被调用的监听器。
```javsacript
const myEmitter = new MyEmitter();

const callbackA = () => {
  console.log('A');
  myEmitter.removeListener('event', callbackB);
};

const callbackB = () => {
  console.log('B');
};

myEmitter.on('event', callbackA);

myEmitter.on('event', callbackB);

// callbackA removes listener callbackB but it will still be called.
// Internal listener array at time of emit [callbackA, callbackB]
myEmitter.emit('event');
// Prints:
//   A
//   B

// callbackB is now removed.
// Internal listener array [callbackA]
myEmitter.emit('event');
// Prints:
//   A
```
这两个方法会改变监听器队列的索引，因此会影响到 `emitter.listeners()`方法。

###emitter.removeListener(eventName, listener)/removeAllListeners([eventName])
`removeAllListeners()`将解除给定事件列表上所有的监听器绑定
`removeListener()` 将解除指定事件列表上指定监听器的绑定，如果同一个监听器被绑定多次到同一个事件上，每次调用方法，只会解除一次绑定，如若要解除多次，就需要多次调用
```javsacript
const callback = (stream) => {
  console.log('someone connected!');
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback);
```

###方法emitter.setMaxListeners(n)/.getMaxListeners()和属性EventEmitter.defaultMaxListeners
Node.js默认为单个特定的事件上最多绑定10个监听器，可以通过`emitter.setMaxListeners(n)`方法为单个特定事件重新设置最大可绑定数量，如果n是一个负数，会抛出TypeError异常。`defaultMaxListeners`是全局所有特定事件最大可绑定的监听器数量，即10，要注意更改此值会影响到全局内所有的事件。不过`setMaxListeners(n)`的优先级高于`defaultMaxListeners`，即后者的改变不影响那些已经使用了前者的事件。
```javascript
emitter.setMaxListeners(emitter.getMaxListeners() + 1);
emitter.once('event', () => {
  // do stuff
  emitter.setMaxListeners(Math.max(emitter.getMaxListeners() - 1, 0));
});
```
###其他
+ `on()`有个别名`addListener()`
+ `emitter.emit('eventName'[,...args])`:该方法可以同时发生多个事件 
+ 以上的方法很多都返回`EventEmitter`，以便链式调用，如`on()`

