##### 1.new操作符到底做了什么？

   1）创建一个新的对象

   2）将构造函数的作用域赋值给新对象

   3）执行构造函数的代码（给新对象添加属性）

   4）返回新对象

#####  2.手写new

```javascript
function _new(fn, ...args) {
    // 创建一个以fn原型为原型的对象
    const obj = Object.create(fn.prototype);
    const result = fn.apply(obj, args); // 将构造函数的作用域赋值给新对象 且 执行构造函数的代码
    // 返回新对象 如何fn返回的是null或undefined(也就是不返回内容) 返回obj 否则返回result
    return result instanceof Object ? result : obj;
}
```

```javascript
/*
以下为测试代码
*/
function Dog(name) {
    this.name = name
}
Dog.prototype.sayName = function() {
    console.log(this.name)
}
var dog = _new(Dog,'小明')
dog.sayName() // 输出结果：小明
```

