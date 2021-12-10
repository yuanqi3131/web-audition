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
<<<<<<< HEAD
function Person(name) { // 提供父类
    this.name = name;
    this.sayName = function () {
        console.log(this.name)
    }
}

Person.prototype.age = 20;

function container(obj) {
    function F() {}
=======
var person = {
    name: 'Bob',
    age: 21,
    study: ['math','subject'],
}

function container(obj) { // Object.create()的原理 Object.create()的参数为新创建对象的原型
    function F() { } // 创建临时性的构造函数
    F.prototype = obj;// 讲传入对象作为临时构造函数的原型
    return new F();// 返回临时构造函数新实例
}

var per = container(person);
console.log(per.name) // Bob
console.log(per.age) // 21
console.log(per.study) // ["math", "subject"]
per.name = 'Tom'
per.age = 22
per.study.push('Chinese')

var per1 = container(person);
console.log(per1.name) // Bob
console.log(per1.age) // 21
console.log(per1.study) // ["math", "subject","Chinese"]

```


缺点： 父类包含引用类型的属性值子类始终都会**共享相应的值** 

##### 5.寄生式继承

```javascript
var person = {
    name: 'Bob',
    age: 21,
    study: ['math','subject'],
}
person.prototype.sayHi = function() {
    console.log(123)
}
function container(obj) {
    function F() { }
>>>>>>> afc73090c4f78e7d2df71020eb93d915398e2d99
    F.prototype = obj;
    return new F();
}

<<<<<<< HEAD
var man = new Person();
var man1 = container(man);
console.log(man1.age); // 20

```

优点：
缺点：

##### 5.寄生式继承
=======
function subContainer(obj) {
    var sub = container(obj);
    sub.address = "中国";
    return sub;
}
var per = subContainer(person);

console.log(per.name) // Bob
console.log(per.age) // 21
console.log(per.study) // ["math", "subject"]
console.log(per.address) // 中国
console.log(per.sayHi()) // 报错 Cannot set property 'sayHi' of undefined
```


缺点：通过上面的报错 我们可以看到没有使用到原型

##### 6.寄生组合式继承
>>>>>>> afc73090c4f78e7d2df71020eb93d915398e2d99

```javascript
function Person(name) { // 提供父类
    this.name = name;
<<<<<<< HEAD
=======
    this.study = ['math','subject']
>>>>>>> afc73090c4f78e7d2df71020eb93d915398e2d99
    this.sayName = function () {
        console.log(this.name)
    }
}

<<<<<<< HEAD
Person.prototype.age = 20;
function container(obj) {
    function F() {}
=======
function container(obj) {
    function F() { }
>>>>>>> afc73090c4f78e7d2df71020eb93d915398e2d99
    F.prototype = obj;
    return new F();
}

<<<<<<< HEAD
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
=======
function subContainer(superType,subType) { //superType：传递父类参数 subType：传递子类参数
    var sub = container(superType.prototype); //相当于Object.create 创建以父类原型为原型的对象
    sub.constructor = subType; // 新创建的对象的构造指向子类
    subType.prototype = sub;// 子类的原型为新创建的对象
}

function Sub(name,age) {
    // 子类
    Person.call(this,name)
}

subContainer(Person, Sub);

var sub1 = new Sub('小潘')
var sub2 = new Sub('小袁')
sub1.sayName(); //小潘
sub2.sayName(); //小袁
sub1.study.push("chinese"); 
console.log(sub1.study) // ["math", "subject", "chinese"]
sub2.study.push("art"); 
console.log(sub2.study) // ["math", "subject", "art"]
```

优点：解决了原型式继承与寄生式继承的缺点，可以访问到原型链上的方法属性，单个实例改变父类的属性方法不会影响到其他实例。

缺点：实现太复杂



7.ES6 class extends继承

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

class man extends Person {
  constructor(x, y) {
    super(x, y); // super一定要放在这里的第一行
  }
}
```

缺点：兼容性问题

 ES5和ES6继承区别就是在于：
　　　　1.ES5先创建子类，在实例化父类并添加到子类this中
　　　　2. ES6的继承是先创建父类的实例对象this(必须先调用super方法), 再调用子类的构造函数修改this 
>>>>>>> afc73090c4f78e7d2df71020eb93d915398e2d99
