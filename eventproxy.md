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






