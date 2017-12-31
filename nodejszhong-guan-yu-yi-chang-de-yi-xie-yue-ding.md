传统的`try/catch/final`语句捕捉异常，在异步编程中是不适用的，如:
```javascript
try{
    JSON.parse(json)
}catch(e){
    //TODO
}
```
这是因为异步的实现包括请求和处理两个阶段，中间夹着个事件循环的执行，如果在第一个阶段提交请求立即返回没有发生异常，而在监听器的处理中发生了异常，那么`try/catch`将不会发生效果。

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

