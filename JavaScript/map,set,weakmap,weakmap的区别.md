### Map,Set,WeakMap,WeakSet的区别

##### 1. Set与WeakSet存在三个区别：

​		1）**WeakSet的成员只能是对象，不能是其他类型的值**

```javascript
var ws = new WeakSet();
ws.add(1); // Invalid value used in weak set
ws.add(null); // Invalid value used in weak set
ws.add(Symbol()); // Invalid value used in weak set
const arr = [3,4];
ws.add(arr); //
ws.has(arr); // true

var s = new Set();
s.add(1); 
s.has(1); // true
s.add(arr);
s.has(arr); // true

```

​		2) **WeakSet没有size属性，不能遍历，Set有size属性可以遍历**。造成WeakSet不能遍历的原因是其成员都是弱引用，即垃圾回收机制不考虑对WeakSet对象的引用，成员随时都会消失，遍历机制无法保证成员的存在，所以根据这一机制，WeakSet的一个用处是储存DOM节点，不用担心DOM节点被删除，而引发内存泄漏

```javascript
var ws = new WeakSet();
ws.size; // undefined
ws.forEach; // undefined
ws.clear; // undefined
```

​		3）WeakSet只有**add**，**delete**，**has**三个方法，Set不仅有这三个方法，还多了clear方法，清除所有值

##### 2. Map与WeakMap存在三个区别（同上面的Set与WeakSet的区别）

​		WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名

```javascript
var map = new WeakMap();
map3.set(1,1); // Invalid value used as weak map key
map3.set({},1); // 成功 
```



##### 3.Map和Set的区别

​		1)  **Set类似于数组，但是成员的值都是唯一的**。Set内部判断两个值是否不同，类似于===，与===的区别在于，===认为NaN === NaN是false，而Set认为NaN和NaN是同一个值，即：

```javascript
var set = new Set();
set.add(NaN);
set.add(NaN);
set.size; // 1
NaN === NaN; // false
```

​		2) **Map类似于对象，本质上是键值对的集合**。但是不同于一般的对象，键只能是字符串，Map的键可以是字符串也可以是对象

```javascript
var map = new Map();
var obj = {name: 1};
map.set(obj, 123);
map.get(obj); // 123
// 注：map的键是采用跟Set一样的，类似于 === ，NaN视为一个键
map.set(NaN, 456);
map.set(NaN, 4567);
map.size(); // 3 但是这里注意4567已经把456替换掉了
```



##### 4.扩展：

​        1）对象转为 Map

```javascript
let obj = {"a":1, "b":2};
let map = new Map(Object.entries(obj)); // Object.entries方法返回一个给定对象自身可枚举属性的键值对数组:[['a',1],['b',1]]
```

​		2) WeakRef

​			ES2021提供，用于直接创建对象的弱引用

​	

```javascript
let obj = {};
let wr = new WeakRef(obj);
wr.deref(); // 如果原始对象存在，该方法返回原始对象 如果原始对象已经被垃圾回收机制清除，该方法返回undefined
```



参考文档：https://es6.ruanyifeng.com/#docs/set-map（阮一峰ES6入门）