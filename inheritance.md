# JavaScript中的继承

标签（空格分隔）： 继承类型 

---
##  继承

### 1. 类式继承 

```javascript
function Person(name, age){
   this.name = name;
   this.age = age;
}
Person.prototype.say = function(){
    console.log('name:' + name + ' age:' +  age);
}
function Worker(name, age, job){
    this.job = job;
}
// 继承Person
Worker.prototype = new Person();
Worker.prototype.sayJob = function(){
    console.log('job:' + this.job);
}
var p1 = new Worker('p1',22,'coder');
p1.sayJob();//'job:coder'
cnosole.log(p1.name);//undefined
```
`Worker`继承了`Person`,而继承是通过创建`Person`的实例并将其赋值给`Worker.prototype`实现的。实现的本质是重写原型对象，代之以一个新类型的实例。于是，`Worker.prototype`不仅拥有`Person`实例所具有的全部属性和和方法，其内部还有一个指针`[[prototype]]`指向了`Person.prototype`。具体如下。
```javascript
 (new Person()).__proto__ == Person.prototype
 Worker.prototype == new Person()
 Worker.prototype.__proto__ == Person.prototype
```

    
    缺陷：
        1.父类中的共有属性如果是引用类型，就会在子类中被所有实例共用，因此若一个子类的实例更改了子类型继承而来的共有属性就会直接影响到其他子类。
        2.无法在不影响所有对象实例的情况下，给父类构造函数传递参数。
    
    Note：所有引用类型都默认继承了Object.

### 2. 构造函数继承
在子类构造函数的内部调用超类构造函数实现继承。解决了包含引用类型值的原型属性会被所有实例共享的问题,并且子类可以向父类构造函数传参。下面代码以说明此问题。
```javascript
//类式继承
function Person(){
    this.name = ['Li', 'Yi'];
}
function Worker(){}
Worker.prototype = new Person();
var p1 = new Worker();
console.log(p1.name);//['Li','Yi']
p1.name.push('Wu');
var p2 = new Worker();
console.log(p2.name);//['li','Yi','Wu']
//构造函数继承
function Person(age){
    this.name = ['Li', 'Yi'];
    this.age = age;
}
function Worker(age, job){
    Person.call(this, age);//继承Person 同时传参
    this.job = job;
}
var p1 = new Worker(22, 'cleaner');
console.log(p1.name);//['Li','Yi']
p1.name.push('Wu');//['Li','Yi','Wu']
console.log(p1.age);//22
console.log(p1.job);//cleaner
var p2 = new Worker(33, 'teacher');
console.log(p2.name);//['li','Yi']
```
由于`call`这个方法可以更改函数的作用环境，因此在子类中对`Person`调用此方法，就是将子类中的变量在父类中重新执行一遍，这样一来`Worker`的每个实例对象就都具有自己`name`属性的副本了。

    缺陷：
        这种类型的继承并未涉及prototype,因此父类的原型方法自然不会被子类继承。如果想要继承父类方法就必须将方法全部置于父类构造函数中，这样每个实例对象都将拥有自己的一套方法而不能共用，函数复用也就无从谈起。
### 3. 组合继承
结合类式继承和构造函数继承二者的优点组合实现继承。
```javascript
function Person(age){
    this.name = ['Li', 'Yi'];
    this.age = age;
}
Person.prototype.sayAge = function(){
    console.log('age:' + this.age);
}
function Worker(age, job){
    Person.call(this, age);//构造函数继承 第二次调用Person()
    this.job = job;
}
Worker.prototype = new Person();//类式继承 第一次调用Person()
Worker.prototype.sayJob = function(){
    console.log('job:' + this.job);
}
var p1 = new Worker(22, 'cleaner');
var p2 = new Worker(33, 'coder');
p1.name.push('Wu');
p1.name;//['Li', 'Yi', 'Wu']
p2.name;//['Li', 'Yi']
p1.sayAge();//age:22
p2.sayJob();//job:cleaner
```
这一继承类型既解决了前面引用类型共享和父类传参的问题，也较好地实现了函数复用。但我们在继承过程中，使用构造函数继承调用了一次父类构造函数，实现子类原型的类式继承时再一次调用了父类构造函数，父类构造函数调用两遍造成性能消耗。实际上我们还可以将继承更简化。
### 4. 原型式继承
借助原型基于已有的对象创建新对象，同时不必创建自定义类型。这种继承类似于前面讲过的类式继承，区别是将子类变成了无内容的临时函数，因此开销较小。但也同时具有类式继承的缺陷。
```javascript
function object(o){

    function F(){} //创建一个临时构造函数
    F.prototype = o; //将传入的对象o 作为构造函数的原型   
    return new F(); //返回临时类型的新实例
}

var Person = {
    name: ['Li', 'Yi']
};
var p1 = object(Person);
var p2 = object(Person);
p1.name;//['Li','Yi']
p1.name.push('Wu');
p2.name;//['Li', 'Yi', 'Wu']
Person.name;//['Li', 'Yi', 'Wu']
```
ECMAScript5通过新增 [Object.create()][1] 方法规范化了原型式继承。
### 5. 寄生式继承
寄生式继承是对原型式继承的二次封装，并在这二次封装过程中对继承的对象进行了拓展。
```javascript
var Person = {
    age: 12
};
function createPerson(o){
    var p = object(o);//利用原型式继承创建新对象
    p.sayAge = function(){//拓展新对象
        console.log();
    };
    return p;//返回拓展后的对象
}
var p1 = createPerson(Person);
p1.sayAge();//12
```
此方式与构造函数继承类似，不能做到函数复用而降低了效率。
### 6. 寄生组合式继承
组合了寄生式继承和组合继承的优点，本质上是使用构造函数继承属性，类式继承继承方法。
```javascript
function inheritPrototype(subClass, superClass){
    var proto = object(superClass.prototype); //创建父类原型副本
    proto.constructor = subClass; //修正构造器指向
    subClass.prototype = proto; //设置子类原型
}

function Person(age){
    this.age = age;
}
Person.prototype.sayAge = function(){
    console.log('age:' + this.age);
};
function Worker(age,job){
    Person.call(this, age);
    this.job = job;
}
inheritPrototype(Worker, Person);
Worker.prototype.sayJob = function(){
    console.log('job:' + this.job);
};
var p1 = new Worker(12, 'student');
p1.sayAge();//age:12
p1.sayJob();//job:student
```
这种继承方式只调用了一次父类构造函数，效率高而且不破坏原型链。


----------


## 2. 拓展
### 1. 作用域安全的构造函数
```javascript
function Person(name, age){
    this.name = name;
    this.age = age;
}
var p1 = new Person('a',11);
p1.name;//"a"
var p2 = Person('b',21);
p2.name;//TypeError: Cannot read property 'name' of undefined
window.name;//"b"
```
当没有用`new`调用构造函数而是直接调用时，由于该`this`对象是在运行时绑定的，因此this会映射到全局对象`window`上，导致错误对象属性的意外增加。

#### **new 操作**
使用`new`来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

```var p = new Person();```

1. 创建全新对象`p`
2. 这个新对象会被执行`[[prototype]]`连接，`p._proto_ == Person.prototype`
3. 这个新对象会绑定到函数调用的`this`
4. 如果函数没有返回其他对象，那么`new` 表达式中的函数调用自动返回当前新对象

#### **this 规则**

1. 函数`new`调用，`this`绑定新创建的对象。`var bar = new foo(); `
2. 函数通过`call`、`apply`、`bind`绑定调用，`this`绑定指定对象。`var bar = foo.call(obj);`
3. 函数在某个上下文对象中调用，`this`绑定当前上下文对象。`var bar = obj.foo();`
4. 如果上述皆不是，使用默认绑定。如果处于严格模式，绑定到`undefined`，否则绑定到全局对象`window`。 `var bar = foo();`

为解决这个问题我们可以创建一个作用域安全的构造函数。调用Person()时无论是否使用new操作符，都会返回一个Person的新实例，这就避免了在全局对象上意外设置属性的情况发生。
```javascript
function Person(name){
    if(this instanceof Person){
        this.name = name;
    }else{
        return new Person(name);
    }
}
```
但这个作用域安全的构造函数存在一个问题，使用构造函数继承时（不涉及原型链）继承将遭到破坏。
```javascript
function Worker(name, job){
    //这里的this对象并非Person的实例 返回的新对象new Person()也没有使用
    Person.call(this, name);
    this.job = job;
}
var p = new Worker('Li',22);
p.name;//undefined
```
如果构造函数继承结合类式继承（原型链）则可以解决这个问题。
```javascript
function Worker(name, job){
    Person.call(this, name);
    this.job = job;
}
Worker.prototype = new Person();
var p = new Worker('Li',22);
p.name;//"Li"
```
实现原理如下
```javascript
new Person() .__proto__ == Person.prototype
Worker.prototyoe.__proto__ == Person.prototype
p.__proto__ == Worker.prototype
P.__proto__.__proto__ == Person.Prototype // this instanceof Person == true
```
顺带提一下`instantof`的实现。实际上是顺着原型链向上查找对应原型，找到返回`true`，反之`false`。
```L.__proto__.__proto__....__proto__ === R.prototype```
```javascript
// L:运算符左表达式 R:右表达式
function instanceOf(L, R){
    var o = R.prototype;
    L = L.__proto__;
    while(true){
        if(L === null)
            return false;
        if(L === o)
            return true;
        L = L.__proto__;
    }
}
```


  [1]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create