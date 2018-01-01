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

###Node.js异步调用的方法嵌套过深
如在网页渲染过程中，通常需要数据、模板、资源文件，三者之间没有依赖关系，但是三者又缺一不可。
我们可能会这样做：
```javascript
fs.readFile(template_path,'utf8',function(err,template){
    db.query(sql, function(err, data){
        l10n.get(function(err, resources){
            //TODO
        });
    });
});
```
结果没有啥问题，但是明显嵌套过深，没有利用好Node.js的异步特性。

###解决嵌套过深，和无法有效利用异步特性的缺陷
为了解决以上问题，我们把所有的事件先拆开，而当这些事件的回调处理完毕时进行页面的渲染。
```javascript
//统计事件完成的个数
var count = 0;
//存放每个事件监听器执行的结果
var results = {};
var done = function(key,value){
    results[key] = value;
    count++;
    //如果做完三件事，那么进行渲染
    if(count === 3) {
        render(results)
    }
};

//被拆开的三个事件
fs.readFile(template_path,"utf-8",function(err,template){
    done("template",template);
});
db.query(sql,function(err,data){
    done("data", data);
});
l10n.get(function(err,resources){
    done("resources",resources);
});
```
这样做很不错，但是有时一个函数执行所需要的完成事件个数不确定，我们需要定义一个哨兵变量用于检测次数。再结合_偏函数_，那么我们可以把`done()`函数中可能会发生变化的_`3`_定义成变量。
```javascript
var after = function(timer, callback){
    var count = 0, results = {};
    return function(key, value) {
        results[key] = value;
        count++;
        if(count === times) {
            callback(results);
        }
    }
}

var done = after(times, render);
```
以上不但解决了Node.js本身的问题，还可以发现在一个函数上关联了多个事件，这就是多对一。
而`发布/订阅`的事件驱动模型，则是一个事件上注册多个回调函数，这是一对多。
把这两种合并下，就是多对多的情况，如下:
```javascript
const EventEmitter = require('events');
const emitter = new EventEmitter();

var after = function(timer, callback){
    var count = 0, results = {};
    return function(key, value) {
        results[key] = value;
        count++;
        if(count === times) {
            callback(results);
        }
    }
}
var done = after(times,render);
emitter.on("done",done);
emitter.on("done",other);
fs.readFile(template_path,"utf-8",function(err,template){
    emitter.emit("done","template",template);
});
db.query(sql,function(err,data){
    emitter.emit("done","data", data);
});
l10n.get(function(err,resources){
    emitter.emit("done","resources",resources);
});
```

根据以上的设计，整理整理，就变成了一个叫`EventProxy`的模块




















