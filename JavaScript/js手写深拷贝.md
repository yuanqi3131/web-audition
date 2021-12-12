### JS手写深拷贝

深拷贝：假设我有一个对象A，我赋值给B，无论怎么改变B，A都不会发生任何的改变，即为深拷贝

浅拷贝：假设我有一个对象A，我赋值给B，改变一下B，A就会跟着改变，A，B对象的地址指向一个地方，只是简单的赋值，即为浅拷贝。

网上有很多说数组的slice、concat方法对数组使用，或者对象使用Object.assign()和ES6的扩展运算符(...)可以实现深拷贝，这是不对的，以上的方法只能对一层的对象有用，多层嵌套的是不起作用的，什么是多层嵌套，下面是两层，obj对象属性name也是一个对象，还可以无限嵌套，以上的方法对下面的多层嵌套是没用的。

var obj = {
    age: 13,
    name: {
        addr: '天边'
    }
}

下面两种方法实现了深拷贝，不管你是一层还是多层对象。

##### 第一种：脑残法

```javascript
function deepClone(obj) {
    if(typeof obj != 'object') return obj;
    return JSON.parse(JSON.stringify(obj))
}

// 测试
var obj = {
    name: 123
}
var obj1 = obj;
obj1.name = 345;
console.log(obj.name); // 345 这里看到 我们把obj赋值给obj1 然后修改obj1的name值，obj的name值同样发生了改变
var obj2 = deepClone(obj);
obj2.name = 678
console.log(obj.name); //345 把obj赋值给obj2 然后修改obj2的name值 obj的值不发生改变
```

##### 第二种：递归法

```javascript
function deepClone(obj) {
    if (typeof obj != 'object') return obj;
    var temp = Array.isArray(obj) ? [] : {};
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            if (obj[key] && typeof obj[key] == 'object') { // 如果obj[key]还是对象则执行递归
                temp[key] = deepClone(obj[key]); // 递归
            } else {
                temp[key] = obj[key];
            }
        }
    }
    return temp;
}

var obj = {
    age: 13,
    name: {
        addr: '天边'
    }
}

var obj2 = deepClone(obj);
obj2.age = 14
obj2.name.addr = '地心'
console.log(obj.age); //13
console.log(obj.name.addr); //天边
```

