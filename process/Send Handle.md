```javascript
//worker.js
var http = require('http');
http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type':'text/plain'});
    res.end('Hello World\n');
}).listen(8888,'127.0.0.1');
```
```javascript
var fork = require('child_process').fork;
var cpus = require('os').cpus();
for(var i=0;i<cpus.length;i++){
    fork('./worker.js');
}
```