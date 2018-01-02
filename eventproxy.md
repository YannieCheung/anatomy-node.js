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











