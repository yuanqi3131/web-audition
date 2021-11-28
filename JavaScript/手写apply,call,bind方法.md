#### 1.手写call方法

```javascript
Function.prototype.customCall = function(context) {
    
        if(typeof this !== 'function') { // 如果不是函数调用 返回异常
            throw new TypeError('Not a Function');
        }

        context = context || window; // 如果没有传递指向对象，则默认为window

        context.fn = this; // 使用fn暂存this

        let args = Array.from(arguments).slice(1); // 获取传递参数值

        let res = context.fn(...args); // 调用函数

        delete context.fn; // 删除变量

        return res; // 返回结果
        
    }
```



#### 2.手写apply方法

```javascript
Function.prototype.customApply = function(context) {
        if(typeof this !== 'function') { // 如果不是函数调用 返回异常
            throw new TypeError('Not a Function');
        }

        context = context || window; // 如果没有传递指向对象，则默认为window

        context.fn = this; // 使用fn暂存this
        let res;
        if(arguments[1]) {
            res = context.fn(...arguments[1])
        } else {
            res = context.fn()
        }
        delete context.fn;

        return res;
    }
```



#### 3.手写bind方法

```javascript
Function.prototype.myBind = function(context) {
        if(typeof this !== 'function') {// 如果不是函数调用 返回异常
            throw new TypeError('Not a Function');
        }

        context = context || window;

        let self = this;

        let args = Array.from(arguments).slice(1);

        return function() {
            return self.call(context,...args)
        }
    }
```

