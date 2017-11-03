# Generator函数

## 基本概念

> 1. function* 声明方式会定义一个生成器函数 Generator 函数，这类函数返回一个 Genertor 对象。
> 
> 2. 这个对象叫做生成器对象，它可以依次遍历 Generator 内部的每个状态。
> 3. 而这一状态是在函数内部用使用 yield 语句定义的。


因此我们可以将 Generator 函数理解为一个遍历器对象生成函数。

```javascript
// 函数拥有三个状态 fir sec return语句
function* generatorFn(){ 
  yield 'fir'; 
  yield 'sec'; 
  return 'end'; 
}

// 函数调用后返回一个指向内部状态的指针对象
// 也就是上面提到过的遍历器对象
var g = generatorFn();

// 遍历器对象的next()方法 可以使指针不断指向下一个状态
// 直至遇到return语句为止
g.next();// {value: "fir", done: false}
g.next();// {value: "sec", done: false}
g.next();// {value: "end", done: true}
g.next();// {value: undefined, done: true}
```
**简言之，Generator 函数是分段执行的，yield 语句是暂停执行的标记，next 方法可以恢复函数执行。**
    
    注意：
        1. Generator 函数中不包含 yield 语句时，只是一个单纯的暂缓执行函数。当调用 next() 方法时才会执行。
        2. yield 不能用在普通函数中否则报错。

## 实例方法
### Generator.prototype.next
`yield`语句无返回值`(undefinded)`，`next()`可以带一个参数这个参数会被当作上一条`yield`语句的返回值。
```javascript
function* g(){ 
  var a = yield 1; 
  var b = yield (a + 1); 
  return b;
}

var f = g();

// 1.无参数情况
f.next(); //Object {value: 1, done: false}
f.next(); //Object {value: NaN, done: false}
f.next(); //Object {value: undefined, done: true}

// 2. 有参数情况
f.next(); //Object {value: 1, done: false}
f.next(1); //Object {value: 2, done: false}
f.next(2); //Object {value: 2, done: true}
```
    注意：第一个 next() 方法用来启动遍历器对象，不需要参数。如果传参会被忽略。
    
**for...of**
`for...of` 自动遍历 `Generator` 函数，不需要借助 `next()`。
```javascript
function* g(){ 
  yield 1; 
  yield 2; 
  yield 3; 
  return 4; 
}

for(let v of g()){
  console.log(v);
}

// 1 2 3
```
    注意：
        一旦 next() 返回对象的属性 done 为 true，循环就会终止，并且不会返回该对象。所以循环并没有得到 return 的返回值4。
```javascript
// 利用Generator函数实现for...of遍历对象
function* objectEntries(){
  let keys = Object.keys(this);
  for(let k of keys){
    yield [k, this[k]];
  }
}

var obj = { name: 'zz', age: 22 };

// 将Generator函数加在对象的Symbol.iterator属性上
obj[Symbol.iterator] = objectEntries;

for(let [k, v] of obj){
  console.log(`${k}:${v}`);
}

// name: zz
// age: 22
```
### Generator.prototype.throw
`throw()` 方法用来向生成器抛出异常，并恢复生成器的执行，返回带有 `done` 及 `value` 两个属性的对象。
```javascript
function* gen() {
  while(true) {
    try {
       yield 1;
    } catch(e) {
      console.log("Error caught!");
    }
  }
}

var g = gen();
g.next(); // { value: 1, done: false }
g.throw(new Error("wrong")); // "Error caught!"
```
### Generator.prototype.return
返回传入的参数值并结束生成器。
```javascript
function* g(){ 
  yield 1; 
  yield 2; 
  yield 3; 
  return 4; 
}

var f = g();

f.next();// Object { value: 1, done: false}
f.next();// Object { value: 2, done: false}
f.return(0);// Object { value: 0, done: true}
f.next();// Object { value: undefined, done: true}
```
    注意：
        1. 调用return()方法后遍历生成器结束，返回值的done属性变为true。
        2. return()不传参返回值为undefined。


## 应用
### yield* 
用来在一个`Generator`函数里面执行另一个`Generator`函数。等同于在生成器函数内部执行一个`for...of`循环，也可以把`yield*`看作递归。
```javascript
function* f(){
  yield 2;
  return 'str';
}
function* g(){
  yield 1;
  var s = yield* f();
  console.log(s);
  yield 3;
}

var t = g();

t.next(); // Object {value: 1, done: false}
t.next(); // "str"
          // Object {value: 2, done: false}
t.next(); // Object {value: 1, done: false}
```
任何数据结构只要有遍历接口就可以用`yield*`遍历。包括数组和字符串等。
```javascript
// 应用：yield*实现数组扁平化 

var flat = function* (arr){
  var len=arr.length;
  for(var i=0;i<len;i++){
    var item = arr[i];
    if(item instanceof Array){
      yield* flat(item);
    }else{
      yield item;
    }
  } 
};

var arr = [1, ["a"], [3, [[4]]]];

for(var i of flat(arr)){
  console.log(i);
}

//1 "a" 3 4
```
### Generator + Promise
```javascript
// 返回 Promise 对象
function print(n){
  return new Promise(function(resolve, reject){
    setTimeout(() => {
      resolve(n);
    })
  });
}

// 在 Generator 函数中书写操作流程
function* start(){
  try{
    var n = yield print(10);
    console.log(n);
  } catch (e) {
    console.log(e);
  }
}

// 函数 run 处理 Promise 对象并在内部调用 next 方法
function run (generator) {
  var it = generator();

  function go(result) {
    if (result.done) return result.value;

    return result.value.then(function (value) {
      return go(it.next(value));
    }, function (error) {
      return go(it.throw(error));
    });
  }

  go(it.next());
}

run(start);

> 10
```
我们使用 `Generator + Promise` 来实现流程管理，读完本文你会发现 `async/await` 其实是他们的的语法糖。
## Async/Await

### 定义
>  **MDN**:
> async 函数声明定义一个异步函数，返回 AsyncFunction 对象。调用 async 函数时会返回一个 Promise 对象。

> 1. 当 async 函数返回一个值时，Promise 的 resolve 方法将会处理这个值。
> 2. 当 async 函数抛出异常时，Promise 的 reject 方法将处理这个异常值。

### 用法

```javascript
async function start(){
  try {
    await promise(); // 如果promise被拒绝则抛出错误被catch
  } catch (e) {
    console.log('Error:' + e);
  }
}
```

1. async 标识符写在函数定义前，表示这是一个异步函数，即可以在内部使用 await。
2. await 只能写在 async 函数中（async 函数内的回调函数也不可以）且后跟一个 Promise 对象（如果是其它普通值会立即返回无意义）
3. 当 await 某个 Promise 对象时，async 函数暂停执行直至该 Promise 返回结果。暂停过程中不阻塞主线程。

**async 函数将异步代码同步化，去掉了回调函数使代码更具可读性。**

### 并行执行

```javascript
function print(n){
  return new Promise(function(resolve, reject){
    setTimeout(() => {
      resolve(n);
    }, 1000);
  });
}

async function start1(){
  var a = await print(10);
  console.log(a);
  var b = await print(20);
  console.log(b);
}

start1();

// after 1000ms
> 10 
// after 1000ms
> 20 

async function start2(){
  var a = print(10);
  var b = print(20);
  console.log(await a);
  console.log(await b);
}

start2();

// after 1000ms
> 10
> 20
```
`start1` 总耗时 `2s`，每间隔 `1s` 按顺序打印出两个数字。`start2` 则仅用时 `1s`，因为两个 `print` 函数是同时发生的。
因此我们在写 async 函数时，要让 `promise` 并行执行以提高效率。