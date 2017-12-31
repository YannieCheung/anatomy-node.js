传统的try/catch/final语句捕捉异常，在异步编程中是不适用的，如:
```javascript
try{
    JSON.parse(json)
}catch(e){
    //TODO
}
```