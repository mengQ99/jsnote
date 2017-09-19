# Promise

## 含义及用法
`Promise`是一个用来传递异步操作消息的对象，他代表了某个未来才会知道结果的事件（一般是异步操作）。`Promise`有三种状态：

- Pending 正在进行
- Resolved 已成功
- Rejected 已失败
```javascript
var promise = new Promise(function(resolve, reject){
    //异步操作
    if(异步操作成功)
        resolve(value);
    else
        reject(error);
});
```
### Promise 实现图片异步加载
```javascript
function imgLoad(url){
    return new Promise(function(resolve, reject){
        var xhr = new XMLHttpRequest();
        xhr.open('GET', url);
        xhr.onload = function(){
            if (request.status === 200) {
                resolve(request.response);
            } else {
                reject(Error('error code:' + request.statusText));
            }
        }
        
        xhr.onerror = function(){
            reject(Error('error!'));
        }
        xhr.send();
    });
}

imgLoad('Image.jpg').then(function(value){
    ...
}, function(error){
    ...
})
```
## 实例方法

### 1. Promise.prototype.then
`then()`为`Promise`实例添加回调函数，返回新的`Promise`实例。第一个参数是`Resolved`状态的回调函数，第二个参数时`Rejected`状态的回调函数。第二个参数可选。
```javascript
promise.then(function(value){
    //success
}, function(error){
    //error
});
```
    由于then()返回新的Promise实例，因此可以链式调用。
### 2. Promise.prototype.catch
等同于`promise.then(null, reject)`，用于捕捉前一个回调函数运行时的错误。
```javascript
promise.then(function(value){
    //success
}).catch(function(error){
    //error
});
```
### 3. Promise.all

    var p = Promise.all([p1, p2, p3]);

`Promise.all()`返回一个新的`promise`对象。

    1. 该promise对象在参数对象里所有的promise对象都成功的时候才会触发成功。所有参数对象返回值组成一个数组传递给promise对象的回调函数。
    
    2. 一旦有任何一个promise对象失败则立即触发该promise对象的失败。第一个失败的返回值传递给promise对象的回调函数。

### 4. Promise.race
    var p = Promise.race([p1, p2, p3]);
与`Promise.all`方法形式上有些类似，但只要其中一个参数对象的状态改变，`Promise`对象的状态就随之改变。
### 5. Promise.resolve
将现有对象转为Promise对象。
### 6. Promise.reject
返回一个状态为失败的Promise对象，并将返回值传递给对应的处理方法。
## Promise的实现
### 1. 基础实现
```javascript
function Promise(fn){
    var value = null,
        deferreds = []; //将回调函数存入数组
    
    this.then = function(onFulfilled){
        deferreds.push(onFulfilled)
        return this; //链式调用
    };
    
    function resolve(value){
        //避免传入同步函数 resolve 先于 then 执行
        setTimeout(function(){
            deferreds.forEach(deferred){
                deferred(value);
            });        
        }, 0)

    }
    fn(resolve);
}
```
### 2. 引入状态