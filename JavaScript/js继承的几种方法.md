##### 1.原型链继承

```javascript
function Person(name) { // 提供父类
    this.name = name;
    this.sayName = function () {
        console.log(this.name)
    }
}

Person.prototype.age = 20;

// 原型链继承
function Man() {
    this.name = '亚当';
}

Man.prototype = new Person();

var man = new Man();
console.log(man.age); // 20
console.log(man.name); // 亚当

/*
Man.prototype = new Person('123');
var man = new Man();
console.log(man.name); // 亚当
// 上面说明原型链继承是不能传参的
*/
```

优点：1.可继承构造函数原型链上的方法与属性
缺点：1.实例不能给父类构造函数传递参数

​			2.无法实现多继承

​			3.所有实例共享父类实例的属性及方法，父类实例修改了原型链上的属性或方法，所有实例的原型属性及方法都会受到影响

##### 2.构造函数继承

```javascript
function Person(name) { // 提供父类
    this.name = name;
    this.sayName = function () {
        console.log(this.name)
    }
}

Person.prototype.age = 20;
    function Man() {
    Person.call(this,'夏娃'); // 给父类构造函数传递参数，解决原型链继承的第一个缺点
    // XXX.call(this) // 这里的XXX可能是别的父类构造函数，解决原型链继承的第二个缺点，可以实现多继承
    this.age = 21;
}

var man = new Man();
console.log(man.age); // 21
console.log(man.name); // 夏娃
// 以下两个console.log解释了下方的缺点1：实例不是父类构造函数的实例，只是子类的实例
console.log(man instanceof Man); // true
console.log(man instanceof Person); // false
```

优点：1.解决了原型链继承的3个缺点，可以给父类构造函数传参，可以实现多继承,因为没有使用原型链所以也不存在原型链方法属性共享的问题
缺点：1.因为没有使用原型链，所以实例不是父类构造函数的实例，只是子类的实例，所以也导致了只能继承父类的属性与方法，不能继承原型的方法及属性



##### 3.组合继承(原型链与构造函数组合)

```javascript
function Person(name) { // 提供父类
    this.name = name;
    this.sayName = function () {
        console.log(this.name)
    }
}

Person.prototype.age = 20;

function SubType(name) {
    Person.call(this,name); // 调用一次父类构造函数
}
SubType.prototype = new Person(); // 调用两次父类构造函数
var sub = new SubType('夏娃');
console.log(sub.age); // 20
console.log(sub.name); // 夏娃
```

优点：1.继承了原型链继承的优点(可以访问父构造函数的原型链属性与方法)，也继承了构造函数继承的优点(可多继承，可传参，每个新实例引入的构造函数属性与方法私有化)
缺点：1.调用了两次父类构造函数，消耗内存，且子类的构造函数会代替原型上的父类构造函数



##### 4.原型式继承

```javascript
function Person(name) { // 提供父类
    this.name = name;
    this.sayName = function () {
        console.log(this.name)
    }
}

Person.prototype.age = 20;

function container(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}

var man = new Person();
var man1 = container(man);
console.log(man1.age); // 20

```

优点：
缺点：

##### 5.寄生式继承

```javascript
function Person(name) { // 提供父类
    this.name = name;
    this.sayName = function () {
        console.log(this.name)
    }
}

Person.prototype.age = 20;
function container(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}

function subContainer(obj){
    var sub = container(obj);
    sub.name="哈哈";
    return sub;
}
var man = new Person();
var man1 = subContainer(man);

console.log(man1.age); // 20
console.log(man1.name); // 哈哈
```

优点：
缺点：