IPC即进程间通信(Inter-Process Communication)，为的是让不同进程能够互相访问资源并协调工作。实现IPC的方式由命名管道、匿名管道、socket、信号量、共享内存、消息队列、Domain Socket等。Node中实现IPC通道的是管道技术。在Node中管道是一个抽象的称呼，具体实现细节由libuv提供，windows下由命名管道实现，*nix系统由Domain Socket实现。表现在应用层上的IPC只有`message`事件和`send()`方法。下图为IPC创建和实现的示意图。
![](/assets/ipc_a.png)

父进程在创建子进程之前，会创建IPC通道并监听它，然后才真正创建出子进程，并通过环境变量告诉子进程这个IPC通道的文件描述符。子进程在启动时，根据文件描述符连接这个已存在的IPC通道，来完成父子进程连接。
![](/assets/ipc_b.png)

因为IPC通道是命名管道或Domain Socket创建的，因此与网络socket的行为比较类似，都是双向通信。不同的是它们在系统内核中就完成了进程通信，而不经过网络层，非常高效。在Node中，IPC通道被抽象为Stream对象，在调用`send()`时发送数据(类似于`write()`)，接收到消息会触发message消息(类似`data`)。