# CommonJS


## 1. 概述

### 属性/方法
每个文件是一个模块，文件中的函数、变量等是私有的，对其它文件不可见。
如果想要在多个文件分享变量，必须定义为`global`对象的属性。
```javascript
global.flag = true; // flag属性可被所有文件读取
```
`CommonJS` 规范规定，每个模块内部的 `module` 变量代表当前模块。`exports` 属性是对外的接口。加载某个模块就是加载该模块的 `exports` 属性。`require` 方法用来引入外部模块。
```javascript
// add.js
var add = function(a, b){
    return a + b;
}
exports.add = add; //通过exports方法对外输出add函数


//result.js
var exp = require('example');
var res = add(3, 5);
```
### 特点

- 所有代码都运行在模块作用域，不会污染全局作用域。
- Node 对引入过的模块都会进行缓存，以减少二次引入时的开销。
- 模块加载的顺序，按照其在代码中出现的顺序。

### 注意
- 不能在 `exports` 上直接赋值，由于 `exports` 对象通过形参方式传入，直接赋值会改变形参的引用（这里不懂的话看实现部分）。如果实在需要改可以直接赋值在 `module.exports` 上。

```javascript
// exp.js
module.exports = () => {
    console.log('Hello World')
}

require('exp')() // 'Hello World'
```

- 传递给 `require()` 的参数必须是符合驼峰命名的字符串，或者以 `./..` 开头的相对路径或绝对路径，可以没有后缀名。
```
require('./exp') 
```

## 2. 实现

### 实现方式

> JS中模块系统的实现一般有两种方式：

> 　1. 将文件中的代码包裹在一个函数中执行，函数返回想要提供给外界的接口。
> 　2. 同是将代码包裹在函数中，但需要通过传递参数的形式使其修改由外部提供的对象，借此对象来给外界提供接口。

```javascript
// 1
var exports = (function(exports){
    exports.a = 'one'
    exports.fn = () => {}
    return exports
})(exports)

exports.a // 'one'

// 2
var exports = {}

(function(exports){
    exports.b = 'two';
    exports.fn = () => {}
})(exports)

exports.b // 'two'
```

`CommonJS/AMD/CMD` 这几大模块系统使用的皆是第二种方式。

### CommonJS实现
在 `Node` 中，每个文模块都是一个对象，其定义如下：
```javascript
function Module(id, parent){
    this.id = id;
    this.exports = {};
    this.parent = parent;
    if(parent && parent.children)
        parent.children.push(this);
    this.filename = null;
    this.loaded = false;
    this.children = [];
}
```
在编译文件模块代码过程中，`Node` 对获取的 `js` 文件内容进行首尾包装。正常的 `js` 文件会被包装成如下形式。
```javascript
(function(exports, require, module, __filename, __dirname){
    var exp = require('example');
    exports.num = 1;
});
```
这样一来便实现了每个模块间的作用域隔离。另外经函数调用后，模块的 `exports` 属性也被返回给了调用方，给外界提供了接口。
```javascript
var module = {
    exports: {}
};

(function(exports, module, ...){
    exports.add = () => { return a + b }
})(module.exports, module);

exports.add(3, 5); //8
```

## 3. 模块加载机制
### 原理

#### 1. id和路径对应
把模块读入数组，模块路径就是模块的 `id`。

> 模块路径是 Node 在定位文件模块的具体文件时制定的查找策略，具体表现为一个路径组成的数组。

#### 2. 动态创建脚本
通过 `createElement('script')` 和 `appendChild(script)` 创建脚本并添加到文档中，根据路径请求脚本内容。

#### 3. document.currentScript
`document.currentScript` 返回当前正在执行的 `script` 元素。
利用返回元素获取匿名模块的 `id`。

#### 4. 依赖分析
将模块转为字符串，利用正则表达式匹配模块中的依赖文件。

#### 5. 递归加载
分析出模块依赖后，需要按照依赖出现的位置进行递归加载。

#### 6. 缓存机制
为每个模块建立缓存对象。


## 4. ES6的模块规范

`ES6` 模块采用静态化思想，使得编译时就能确定模块间的依赖关系。而 `CommonJS` 和 `AMD` 都只能在运行时确定。

### export - 定义模块

```javascript
// example.js
export var add = function (a, b){
    return a + b;
}
```

### import - 引入模块
```javascript
// result.js
import {add} from 'example';
var res = add(3, 5);
```

    注意：import 会执行所加载的模块。

### export default - 默认输出
```javascript
// defalut.js
export default function foo(){}

// import 指令可以指定任意名字 无需知道所需加载的变量或函数名
import anyName from 'default'
```
### 模块继承
```javascript
export * from 'example'  // export * 表示输出模块的所有变量及函数
```