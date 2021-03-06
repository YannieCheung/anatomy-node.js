传统的`try/catch/final`语句捕捉异常，在异步编程中是不适用的，如:
```javascript
try{
    JSON.parse(json)
}catch(e){
    //TODO
}
```
这是因为异步的实现包括请求和处理两个阶段，中间夹着个事件循环的执行，如果在第一个阶段提交请求立即返回没有发生异常，而在监听器的处理中发生了异常，那么`try/catch`将不会发生效果，更具体的分析如下。

异步方法定义一般如下
```javascript
var async = function(callback) {
    process.nextTick(callback);
}
try{
    async(callback);
}catch(e){
    //TODO
}
```
调用`async(fn)`方法后，回调函数被存储起来，直到`EventLoop`的下一个Tick才拿出来执行，`try/catch`只对当次Tick起到作用，而无法作用下一次的Tick。

Node.js的API中对异常的处理有以下的约定。将异常作为回调函数的第一个参数传回，在回调中判断，如果其为空值，表示没有异常抛出。
```javascript
async(function(err, results){
    if(err) 
        //有异常
    //TODO 没有异常
})
```

而对于我们自己编写的异步方法，需要遵循以下两个原则：
* 在方法的定义中，我们必须执行一次调用者传入的回调函数
* 需要向Node自身API那样，为调用者传递会异常，供其作判断用
```javascript
var async = function(callback){
    process.nextTick(function(){
        var results = something;
        if(error) {
            return callback(error);
        }
        callback(null, results);
    });
};
async(function(error,results){
    if(err)
        //有异常
    //TODO 没有异常
});
```
