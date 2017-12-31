###偏函数
现有如下两个函数，传递一个对象给它们后，它们会判断该对象是否是一个字符串或者函数。
```javascript
var toString = Object.prototype.toString;
var isString = function(obj) {
    return toString.call(obj) === '[object String]';
}
var isFunction = function(obj) {
    return toString.call(obj) === '[object Function]';
}
var func = function(){console.log('我是函数')};
var str = '我是字符串';
console.log(isFunction(func));
//打印 true
```