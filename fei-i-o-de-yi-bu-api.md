###定时器setTimeout()和setInterval()
定时器也是一种异步调用，当其被创建时，会被插入到`EventLoop`体系中的定时器观察者上的一个红黑树中。每次Tick执行时，会迭代出树上的定时器对象，检查这些对象是否超过时间，如果超过，就会形成一个事件，如果JS线程空闲将立即执行回调函数。
![](/assets/timer_event.png)

值得注意的是，当定时器事件发生执行回调前，如果`EventLoop`有其他的事件正在执行，而实际的定时器时间已经超时，也是要等待前面的事件执行完毕才会执行定时器的回调，因此定时器不一定是准时触发回调。

###process.nextTick()和setImmediate()
有时候要立即执行一个任务，可能会这样写
```javascript
setTimeout(function(){},0);
```
因为`setTimeout()`要动用到红黑树，较为浪费性能。
而`process.nextTick()`和`setImmediate()`则较为轻量级，前者的代码如下:
```javascript
//process.nextTick()
process.nextTick = function(callback) {
    if(process._exiting) return;
    if(tickDepth >= process.maxTickDepth)
        maxTickWarn();
    var tock = {callback:callback};
    if(process.domain) tock.domain = process.domain;
    nextTickQueue.push(tock);
    if(nextTickQueue.length) {
        process._needTickCallback();
    }
}
```
可以发现，每次调用`process.nextTick()`只会将回调函数放入队列，在下一次Tick时执行。

看下面的代码
```javascript
setTimeout(function(){
    console.log('setTimeout延迟执行');
},0);
setImmediate(function(){
    console.log('setImmediate延迟执行');
});
process.nextTick(function(){
    console.log('nextTick延迟执行');
});

console.log('正常执行');
//打印
//正常执行
//nextTick延迟执行
//setTimeout延迟执行
//setImmediate延迟执行
```
根据输出，优先级从大到小为，`process.nextTick()`、`setTimeout(fn,0)`、`setImmediate()`，这是因为他们的观察者就不同，`EventLoop`对观察者的检查顺序是有先后的。

以上三个方法如果含有如下这种嵌套，那么执行顺序优先是到底是什么方法，然后才是内层外层。
```javascript
setTimeout(function(){
    console.log('setTimeout延迟执行');
    process.nextTick(function(){
        console.log('强势插入3');
    });
    console.log('---------------------');
},0);
setImmediate(function(){
    console.log('setImmediate延迟执行1');
    process.nextTick(function(){
        console.log('强势插入1');
    });
});
setImmediate(function(){
    console.log('setImmediate延迟执行2');
});
process.nextTick(function(){
    console.log('nextTick延迟执行1');
});
process.nextTick(function(){
    console.log('nextTick延迟执行2');
    process.nextTick(function(){
        console.log('强势插入2');
    });
    console.log('------------------');
    setImmediate(function(){
        console.log('强势插入4');
    });
});
```