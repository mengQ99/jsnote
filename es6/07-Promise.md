# Promise

## 含义及用法
`Promise`是一个用来传递异步操作消息的对象，他代表了某个未来才会知道结果的事件（一般是异步操作）。`Promise`有三种状态，状态不受外界影响且状态一旦改变不可逆转：

- `Pending` 正在进行
- `Resolved` 已成功
- `Rejected` 已失败
```javascript
var promise = new Promise(function(resolve, reject){
    if(success) //异步操作成功时执行
        resolve(value);
    else // 异步操作失败时执行
        reject(error);
});

promise.then(function(val){
    //定义 resolve 方法
}, function(error){
   //定义 reject 方法 
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

### Promise.prototype.then
`then()` 为 `Promise` 实例添加回调函数，返回新的 `Promise` 实例。第一个参数是 `Resolved` 状态的回调函数，第二个参数是 `Rejected` 状态的回调函数。第二个参数可选。
```javascript
promise.then(function(value){
    //success
}, function(error){
    //error
});
```
    由于 then() 返回新的 Promise 实例，因此可以链式调用。
### Promise.prototype.catch
等同于`promise.then(null, reject)`，用于捕捉前一个回调函数运行时的错误。
```javascript
promise.then(function(value){
    //success
}).catch(function(error){
    //error
});
```
### Promise.all

    var p = Promise.all([p1, p2, p3]);

`Promise.all()`返回一个新的`promise`对象。

    1. 该promise对象在参数对象里所有的promise对象都成功的时候才会触发成功。所有参数对象返回值组成一个数组传递给promise对象的回调函数。
    
    2. 一旦有任何一个promise对象失败则立即触发该promise对象的失败。第一个失败的返回值传递给promise对象的回调函数。

### Promise.race
    var p = Promise.race([p1, p2, p3]);
与`Promise.all`方法形式上有些类似，但只要其中一个参数对象的状态改变，`Promise`对象的状态就随之改变。
### Promise.resolve
将现有对象转为Promise对象。
### Promise.reject
返回一个状态为失败的Promise对象，并将返回值传递给对应的处理方法。
## Promise的实现
### 基础实现
```javascript
function Promise(fn){
    var value = null, //[[PromiseValue]]
        queue = [];
    
    this.then = function(onResolved){
        queue.push(onResolved)
    };
    
    function resolve(value){
        queue.forEach((callback) => {
            callback(value);
        });        
    }
    
    fn(resolve);
}
```

1. 定义 `queue` 数组作为回调函数队列，保证回调按序执行。
2. 调用 `then()` 将传入的参数添加到回调函数队列中。
3. 定义 `resolve` 函数，参数 `value` 为异步操作返回结果。队列遍历，回调函数顺序执行。

### 延时
但上述实现存在一个问题。如果 `Promise` 实例中的代码是同步的，那么代码中的 `resolve` 函数（成功回调函数）会立即执行。此时 `then()` 方法注册的回调还未添加到队列中，会导致代码静默失败。
```javascript
function resolve(value){
    setTimeout(function(){
        queue.forEach((callback) => {
            callback(value);
        });        
    }, 0)
}
```
`Promises/A+` 规范明确要求回调需要通过异步方式执行，用以保证一致可靠的执行顺序。因此我们可以使用 `setTimeout` 将执行回调的操作异步执行。


### 链式调用
我们看到 `then()` 是可以链式调用的，但它的链式调用并不是一句 `return this` 那么简单。`Promise/A` 标准中，规定 `then()` 要返回一个新的对象（`Promise/A+` 中将这一条件删去），于是有以下实现。
```javascript
var promise2;

this.then = function(onResolved){
    promise2 = new Promise(function(resolve){
        ...
    })
}
```
### 引入状态
> 一旦状态改变就不会再变，任何时候都可以得到这个结果。

> Promise 对象的改变只有两种可能： **Pending -> Resloved** 和 **Pending -> Rejected** 。

> 只要其中之一发生，状态就凝固了，结果也不会再改变。就算改变已经发生，我们再对 Promise 对象添加回调函数也会立即得到这个结果。 

```javascript
var value = null,
    status = 'pending', //[[PromiseStatus]]
    promise2,
    onResolvedQueue = [], 
    onRejectedQueue = []; 
    
this.then = function(onResolved, onRejected){
    if(status == 'pending'){
        promise2 = new Promise(function(resolve, reject){
            ...
        });
    }
    if(status == 'resolved'){
        promise2 = new Promise(function(resolve, reject){
            ...
        });
    }
    if(status == 'rejected'){
        promise2 = new Promise(function(resolve, reject){
            ...
        });
    }
    return promise2;
};
    
function resolve(val){
    if(status == 'pending'){
        value = val;
        status = 'resolved';
        setTimeout(function(){
            onResolvedQueue.forEach((callback) => {
                callback(value);
            });        
        }, 0);
    }
}
    
function reject(reason){
    if(status == 'pending'){
        value = reason;
        status = 'rejected';
        setTimeout(function(){
            onRejectedQueue.forEach((callback) => {
                callback(value);
            });        
        }, 0);        
    }
}

fn(resolve, reject)

```
`resolve & reject` 函数内部先判断状态是否为 `pending` ，若是将状态改变为 `resolved/rejected`。一旦状态改变，之后通过 `then()` 注册的回调都会立即执行且结果不变。

### 串行Promise
我们常常利用前一个异步操作的结果来获取下一个异步操作的结果，这就是串行 `Promise`。

> [根据标准][1]，promise2 的值取决于 then 方法内回调函数的返回值。如果 onResolved/onRejected 的返回值 x 是一个 Promise， 则使 promise 接受 x 的状态。promise2 的结果将直接取这个 Promise 的结果。

```javascript
...
this.then = function(onResolved, onRejected){
  // 如果 promise 状态已经改变 直接执行 onResolved/onRejected
  if(status == 'resolved'){
    promise2 = new Promise(function(resolve, reject){
      var x = onResolved(value);
      if(x instanceof Promise){
        x.then(resolve, reject);    
      }
      resolve(value)
    });
  }
  if(status == 'rejected'){
    promise2 = new Promise(function(resolve, reject){
      var x = onRejected(value);
      if(x instanceof Promise){
        x.then(reslove, reject);
      }
    });
  }
  //如果 promise 状态还未改变 则需要将两种处理逻辑添加到相应队列中
  if(status == 'pending'){
    promise2 = new Promise(function(resolve, reject){
      onResolvedQueue.push(function(val){
        var x = onResolved(value);
        if (x instanceof Promise) {
          x.then(resolve, reject);
        }
      });
      onRejectedQueue.push(function(reason){
        var x = onRejected(value);
        if(x instanceof Promise){
          x.then(resolve, reject);
        }
      });
    });
  }
  return promise2;
}
...
```

```javascript
//例：首先异步获取用户 name，再根据获取结果异步获取用户 age
function getName(){
    return new Promise(function(resolve, reject){
        setTimeout(() => {
            resolve('qz');
        }, 1000);
    })
}
function getAge(name){
    console.log(name+':');
    return new Promise(fucntion(resolve, reject){
        setTimeout(() => {
            resolve('22');
        }, 1000);
    })
}

getName()
    .then(getAge)
    .then(function(value){
        console.log(value)
    }, function(reason){
        return new Error(reason);
    });

> qz:
> 22
```


  [1]: https://promisesaplus.com/#point-40