# Set 和 Map 数据结构

## Set
### 用法
`Set`类似于数组，成员值唯一不重复。其本身是一个构造函数，用来生成`Set`数据结构。`Set()`仅接受以数组或类数组对象初始化参数。
```javascript
var s = new Set([1, 1, 2, 3, 3, 4]);
s; // Set {1, 2, 3, 4}

var t = new Set([1, '1', NaN, NaN, {}, {}]);
t; // Set {1, "1", NaN, Object {}, Object {}}
```
    注意：
        1. Set内部判断两值是否相等使用严格相等运算符(===)。
        2. 特殊情况是认为NaN等于自身。    
### 属性和方法

- 实例属性
    - `Set.prototype.constructor` : 指向构造函数Set。
    - `Set.prototype.size` : 成员个数，相当于数组的length。

- 实例操作方法
    - `add()` : 添加值返回Set结构本身，因此可以链式操作。
    - `delete()` : 删除值返回布尔值表示是否删除成功。
    - `has()` : 判断一个值是否为Set成员。
    - `clear()` : 清除所有Set成员无返回值。

`Array.from()`方法可以将`Set`结构转为数组。因此我们就有了一种数组去重的新方法。
```javascript
function unique(arr){
    return Array.from(new Set(arr));
}
unique([1, 2, 3, 3, 1]); //[1, 2, 3]
```
- 实例遍历方法
    - `keys()` : 返回一个键名的遍历器。
    - `values()` : 返回一个键值的遍历器。
    - `entries()` : 返回一个键值对的遍历器。
    - `forEach()` : 使用回调函数遍历每个成员。
```javascript
var s = new Set(['a','b','c']);
for(let i of s.entries()){ 
    alert(i);
}
// ["a", "a"]
// ["b", "b"]
// ["c", "c"]
```

> `Set`结构默认可遍历，我们可以使用`for...of`来循环遍历`Set`。因此我们可以使用另一种更方便的方法实现数组去重。
```javascript
var uniq = [...new Set([arr])];
```
## WeakSet

`WeakSet`结构与`Set`类似，是不重复值的集合。但与`Set`存在以下区别。
    
    1. 成员只能是对象否则报错。
    2. 对象成员皆为弱引用，因此WeakSet不可遍历。
    3. 没有size属性。

- 实例方法
    - `add()` : 添加新对象返回WeakSet结构本身
    - `delete()` : 删除对象，返回布尔值表示是否删除成功。
    - `has()` : 判断一个对象是否为Set成员。
```javascript
var ws = WeakSet();
ws.add(1); //TypeError: Invalid value used in weak set
ws.add([1, 2]); //WeakSet {[1, 2]}
```
## Map
### 用法
普通`Object`结构只能接受字符串作为键名，而`Map`结构接受各种类型的值包括对象作为键名。接受数组作为参数。
```javascript
var p = new Map([['a', 1], ['b', 2]]);
p.get(a); // 1

var m = new Map();
var o = { 'a': 1 };
m.set(o, 'one');
m.get(o); // "one"
```
    注意：
        1. 对同一键多次赋值，仅最后一次赋值有效。
        2. 对同一对象的引用才视为同一个键。
### 属性和方法
- 实例属性
    - `Map.prototype.constructor` : 指向构造函数Map。
    - `Map.prototype.size` : 返回成员个数。

- 实例操作方法
    - `set(key, value)` : 设置key对应的键值value，返回整个Map结构。如果key存在键值被更新，否则生成新键值对。
    - `get(key)` : 读取key对应键值，不存在返回undefined。
    - `delete()` : 删除键，返回布尔值表示是否删除成功。
    - `has()` : 判断一个键是否存在于Map中。
    - `clear()` : 清除所有Map成员无返回值。


- 实例遍历方法
    - `keys()` : 返回一个键名的遍历器。
    - `values()` : 返回一个键值的遍历器。
    - `entries()` : 返回一个键值对的遍历器。
    - `forEach()` : 使用回调函数遍历每个成员。
```javascript
var m = new Map([['a', 1], ['b', 2]]);
for(let [k, v] of m){
    console.log(k, v);
}
// a 1
// b 2
```
### Map与其他结构的转换
**Map -> Array**
```javascript
[...new Map([['a', 1], ['b', 2]])];
// [['a', 1], ['b', 2]]
```
**Array -> Map**
```javascript
new Map(arr);
```
**Map -> Object**
```javascript
for(let [k, v] of strMap){
    obj[k] = v;
}
```
**Object -> Map**
```javascript
for(let k of obj){
    strMap.set(k, obj[k]);
}
```
## WeakMap
与Map结构类似，仅接受对象作为键名。另外与WeakSet相似键名指向的对象是弱引用。
    
    1. 无size属性。
    2. 无法遍历且没有clear()方法。