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

现在如果有很多此类判断对象类型功能需要实现，就要写很多此类函数。为此我们找到更好的解决办法。
```javascript
var isType = function(type){
    return function(obj){
        return toString.call(obj) == '[object ' + type + ']';
    }
}
var isFunc = isType('Function');
var isStr = isType('String');
console.log(isFunc(func));
console.log(isStr(str));
//打印 true true
```
我们抽取出这些函数的不同的地方作为变量，在外层包裹一个函数，变量通过包裹函数的参数获取。
如上可得偏函数定义：指定部分变量，在函数外部形成一个新的函数。优点是可合并功能类似的一组函数。
调用时，调用外层函数，为确定变量传入参数，获取特定函数后再次调用。

#Node.js异步调用的方法嵌套过深
如在网页渲染过程中，通常需要数据、模板、资源文件，三者之间没有依赖关系，但是三者又缺一不可。
我们可能会这样做：
```javascript

```





















