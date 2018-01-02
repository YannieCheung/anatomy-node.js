`EventProxy`的基础参见<a href='./event_proxy_base.md?_k=j2nxv3'>多异步协作</a>
`EventPorxy`可以给事件编程带来以下几点思维上的变化。

* 利用事件机制解耦复杂业务逻辑
* 移除被广为诟病的深度callback嵌套问题
* 将串行等待变成并行等待，提升多异步协作场景下的执行效率
* 友好的Error handling
* 无平台依赖，适合前后端，能用于浏览器和Node.js
* 兼容CMD，AMD以及CommonJS模块环境

使用它之前的代码，是深度嵌套并且串行的。
```javascript
var render = function (template, data) {
  _.template(template, data);
};
$.get("template", function (template) {
  // something 
  $.get("data", function (data) {
    // something 
    $.get("l10n", function (l10n) {
      // something 
      render(template, data, l10n);
    });
  });
});
```

使用它之后的代码，是无嵌套并且并行的。
```javascript
var ep = EventProxy.create("template", "data", "l10n", function (template, data, l10n) {
  _.template(template, data, l10n);
});
 
$.get("template", function (template) {
  // something 
  ep.emit("template", template);
});
$.get("data", function (data) {
  // something 
  ep.emit("data", data);
});
$.get("l10n", function (l10n) {
  // something 
  ep.emit("l10n", l10n);
});
```

#异步协作
###多类型异步协作
如下以渲染页面为例，渲染页面需要模板、数据。假定模板和数据的读取都需要作异步操作。
`all()`方法将一个监听器(handler)注册到`tpl`和`data`的事件组合上。当`tpl`和`data`事件都触发后，每个事件传递的参数将会依照`all()`方法的(事件名)参数顺序，传入监听器作为参数，之后执行监听器。
```javascript
var ep = new EventProxy();
ep.all('tpl', 'data', function (tpl, data) { // or ep.all(['tpl', 'data'], function (tpl, data) {}) 
  // 在所有指定的事件触发后，将会被调用执行 
  // 参数对应各自的事件名 
});
fs.readFile('template.tpl', 'utf-8', function (err, content) {
  ep.emit('tpl', content);
});
db.get('some sql', function (err, result) {
  ep.emit('data', result);
});
```
#####快速完成绑定
以下方式等价于`all()`
```javascript
var ep = EventProxy.create('tpl', 'data', function (tpl, data) {
  // TODO 
});
```

###重复异步协作
如下以读取目录下所有文件为例，在异步操作中，我们多次调用相同的操作后，再执行handler。
此时便使用`after()`函数，handler的参数接受一个数组，按照多次触发事件的顺序，传入参数。
```javascript
var ep = new EventProxy();
ep.after('got_file', files.length, function (list) {
  // 在所有文件的异步执行结束后将被执行 
  // 所有文件的内容都存在list数组中 
});
for (var i = 0; i < files.length; i++) {
  fs.readFile(files[i], 'utf-8', function (err, content) {
    // 触发结果事件 
    ep.emit('got_file', content);
  });
}
```
在after的回调函数中，结果顺序是与用户emit的顺序有关。为了满足返回数据按发起异步调用的顺序排列，EventProxy提供了`group()`方法。
```javascript
var ep = new EventProxy();
ep.after('got_file', files.length, function (list) {
    // 在所有文件的异步执行结束后将被执行 
    // 所有文件的内容都存在list数组中，按顺序排列 
});
for (var i = 0; i < files.length; i++) {
    fs.readFile(files[i], 'utf-8', ep.group('got_file'));
}
```
关于异常见下面的异常处理部分，`group()`也类似于下面的`done()`方法
```javascript
ep.group('got_file');
// 约等价于 
function (err, data) {
  if (err) {
    return ep.emit('error', err);
  }
  ep.emit('got_file', data);
};
```

###持续型异步协作
在`all()`方法的使用中，当其中一个事件再次触发时，handler不会再次调用。
而`tail()`方法，在指定事件触发后，如果事件之后会持续的触发，那么handler会在每次触发事件时持续的调用，像条尾巴那样。
比如股票，数据和模板都是异步获取，视图会随着数据的刷新而刷新，如下
```javascript
var ep = new EventProxy();
ep.tail('tpl', 'data', function (tpl, data) {
  // 在所有指定的事件触发后，将会被调用执行 
  // 参数对应各自的事件名的最新数据 
});
fs.readFile('template.tpl', 'utf-8', function (err, content) {
  ep.emit('tpl', content);
});
setInterval(function () {
  db.get('some sql', function (err, result) {
    ep.emit('data', result);
  });
}, 2000);
```

#异常处理
在通常情况下我们都是像下面这样通过添加一个`error`事件来进行处理的

```javascript
exports.getContent = function (callback) {
    var ep = new EventProxy();
    ep.all('tpl', 'data', function () {
        // 成功回调 
        callback(null, {
            template: tpl,
            data: data
        });
    });
    //侦听error事件
    ep.bind('error', function (err) {
        // 卸载掉所有handler 
        ep.unbind();
        // 异常回调 
        callback(err);
    });
    fs.readFile('template.tpl', 'utf-8', function (err, content) {
        if (err) {
            // 一旦发生异常，一律交给error事件的handler处理 
            return ep.emit('error', err);
        }
        ep.emit('tpl', content);
    });
    db.get('some sql', function (err, result) {
        if (err) {
            // 一旦发生异常，一律交给error事件的handler处理 
            return ep.emit('error', err);
        }
        ep.emit('data', result);
    });
}
```
可以发代码量比不处理异常前增加太多，因此通过`fial()`/`throw()`/`done()`等方法进行如下优化
```javascript
exports.getContent = function (callback) {
 var ep = new EventProxy();
  ep.all('tpl', 'data', function (tpl, data) {
    // 成功回调 
    callback(null, {
      template: tpl,
      data: data
    });
  });
  // 添加error handler 
  ep.fail(callback);
 
  fs.readFile('template.tpl', 'utf-8', ep.done('tpl'));
  db.get('some sql', ep.done('data'));
};
```
`fail()`方法侦听了error事件，默认处理卸载掉所有handler，并调用异常回调函数。
```javascript
ep.fail(callback);
// 由于参数位相同，它实际是 
ep.fail(function (err) {
    callback(err);
});
// 等价于 
ep.bind('error', function (err) {
    // 卸载掉所有handler 
    ep.unbind();
    // 异常回调 
    callback(err);
});
```
`done()`方法返回一个函数的定义，因为`Events`的异常处理的最佳实践中，handler的第一个参数必定是个`error`对象。检查到异常后，将会触发`error`事件。剩下的参数，将触发事件，传递给对应的handler处理。
```javascript
ep.done('tpl');
// 等价于 
function (err, content) {
  if (err) {
    // 一旦发生异常，一律交给error事件的handler处理 
    return ep.emit('error', err);
  }
  ep.emit('tpl', content);
}
```
`throw()`也是个简写
```javascript
var err = new Error();
ep.throw(err);
// 实际是 
ep.emit('error', err);
```



