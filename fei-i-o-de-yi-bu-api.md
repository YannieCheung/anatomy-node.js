###定时器setTimeout()和setInterval()
定时器也是一种异步调用，当其被创建时，会被插入到`EventLoop`体系中的定时器观察者上的一个红黑树中。每次Tick执行时，会迭代出树上的定时器对象，检查这些对象是否超过时间，如果超过，就会形成一个事件，如果JS线程空闲将立即执行回调函数。
![](/assets/timer_event.png)

值得注意的是，当定时器事件发生执行回调前，如果`EventLoop`有其他的事件正在执行，而实际的定时器时间已经超时，也是要等待前面的事件执行完毕才会执行定时器的回调，因此定时器不一定是准时触发回调。

###process.nextTick()和setImmediate()
有时候要立即执行一个任务，可能会这样写
```javascript
setTimeout(function(){},0);
```