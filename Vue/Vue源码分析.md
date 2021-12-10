### Vue2.x源码分析(双向绑定 )

##### 1.原理简析

Vue数据双向绑定是通过数据劫持结合发布-订阅者模式来现实的。其中数据劫持是通过Object.defineProperty()来实现，而发布订阅模式也很好理解，类比用户关注公众号，你关注了某个公众号，当这个公众号有内容更新时，所有关注他的人都可以收到他更新的信息，即发布订阅。

简单说一下Object.defineProperty()，看下面代码：

```javascript
var company = {}; // 公司

Object.defineProperty(company, 'department', {
    set: function(value) {
        console.log('部门为：' + value);
        department = value;
    },
    get: function() {
        return '('+ department +')';
    }
})

company.department = '法务部'; // 部门为：法务部
console.log(company.department) // (法务部)
```

通过上面的代码我们可以通过set，get来操控对象属性，这就是数据劫持的意思，白话就是过路我先打劫一顿你再过去，劫持！

##### 2.所用版本

```javascript
/*!
 * Vue.js v2.6.14
 * (c) 2014-2021 Evan You
 * Released under the MIT License.
 */
```

##### 3.源码分析

```javascript
<script>
    var vm = new Vue({ // 我们使用Vue new一个实例
        data() {
            return {
                msg: 'Hello Vue'
            }
        },
        created() {
            console.log(this.msg) // 打印Hello Vue
        }
    })
</script>
```

从上面的代码我们可以在vue.js中找到Vue的入口

```javascript
function Vue (options) {
    if (!(this instanceof Vue)
       ) { // 这里控制一定要使用new一个构造函数创建一个实例 如果你把上面的new Vue({...})的new去掉，你就可以看到控制台报这个错误
        warn('Vue is a constructor and should be called with the `new` keyword');
    }
    this._init(options); // 调用Vue原型上的_init方法
}
```

```javascript
initMixin(Vue); // Vue构造函数下面会调用initMixin方法，initMixin方法里面定义了原型上的_init方法
// 因为我们本章是研究双向绑定的原理，所以我只关注相应逻辑，其他的功能后续文章会分析
Vue.prototype._init = function (options) {
      var vm = this;
      // a uid
      vm._uid = uid$3++;

      var startTag, endTag;
      /* istanbul ignore if */
      if (config.performance && mark) {
        startTag = "vue-perf-start:" + (vm._uid);
        endTag = "vue-perf-end:" + (vm._uid);
        mark(startTag);
      }

      // a flag to avoid this being observed
      vm._isVue = true;
      // merge options
      if (options && options._isComponent) {
        // optimize internal component instantiation
        // since dynamic options merging is pretty slow, and none of the
        // internal component options needs special treatment.
        initInternalComponent(vm, options);
      } else {
        // merge--英文合并融合的意思 mergeOptions合并options 这里的options为 new Vue传递过来的参数
        // 因为Vue方法自带了一些option所以要跟用户传递过来的合并，这一块的知识也后面讲，你现在只要知道这一步是把用户传递过来的参数与Vue构造函数自带             // 的options进行合并并把他们储存到Vue的$options变量里
        vm.$options = mergeOptions(
          resolveConstructorOptions(vm.constructor),
          options || {},
          vm
        );
      }
      /* istanbul ignore else */
      {
        initProxy(vm); // 初始化代理
      }
      // expose real self
      vm._self = vm;
      initLifecycle(vm); // 初始化变量 定义$parent,$root等一些变量
      initEvents(vm); // 初始化事件
      initRender(vm); // 以后补充
      callHook(vm, 'beforeCreate');// 调用beforeCreate hook钩子函数
      initInjections(vm); // resolve injections before data/props 初始化injections
      initState(vm); // 本期双向绑定的重点！！！
      initProvide(vm); // resolve provide after data/props 初始化provide
      callHook(vm, 'created'); // 调用created hook钩子函数

      /* istanbul ignore if */
      if (config.performance && mark) { // 官方API解释如下，开发调试相关
          // 设置为 true 以在浏览器开发工具的性能/时间线面板中启用对组件初始化、编译、渲染和打补丁的性能追踪。
          // 只适用于开发模式和支持 performance.mark API 的浏览器上。
        vm._name = formatComponentName(vm, false);
        mark(endTag);
        measure(("vue " + (vm._name) + " init"), startTag, endTag);
      }

      if (vm.$options.el) { // 如果创建实例的时候没有传递el，停止整个生命周期的流程，直到手动触发执行了vm.$mount(el)
        vm.$mount(vm.$options.el);
      }
    };

```

从上面的注释我们可以很清楚的知道Vue的生命周期，从new Vue() -> initLifecycle(vm)，initEvents(vm) ->callHook(vm, 'beforeCreate')调用钩子函数->initInjections(vm)->callHook(vm, 'created') 调用created hook钩子函数->是否传递了el，没有则停止生命周期，有则继续往下走。对应下面的流程图。

![image-20211209103439860](C:\Users\83891\AppData\Roaming\Typora\typora-user-images\image-20211209103439860.png)

```javascript
function initState (vm) {
    // 双向绑定的核心在于上面的initState()方法，你可能会很疑惑，那么多东西，我都看不懂，但是看源码就是这样的，你不能一口吃成一个胖子
    // 看源码要一部分一部分去看，不断蚕食，才能到最后去了解他整体。本篇文章讲述的是双向绑定，抓住核心，其他的慢慢研究。
    vm._watchers = []; 
    var opts = vm.$options; // 获取options值
    if (opts.props) { initProps(vm, opts.props); }// options有没有props，有则初始化props
    if (opts.methods) { initMethods(vm, opts.methods); }// options有没有methods，有则初始化methods
    if (opts.data) {// options有没有data，有则初始化data
      initData(vm);
    } else { // options没有data 则把一个空对象当作根节点数据
      observe(vm._data = {}, true /* asRootData */);
    }
    // 从下面两个判断可知，Vue是先初始化computed在初始化watch
    if (opts.computed) { initComputed(vm, opts.computed); }// options有没有computed，有则初始化computed
    if (opts.watch && opts.watch !== nativeWatch) {// options有没有watch，有则初始化watch
      initWatch(vm, opts.watch);
    }
  }
```

```javascript
function initData (vm) { // 初始化data
    var data = vm.$options.data; // 获取实例的data 
     // 即data = data() {
        //     return {
        //         msg: 'Hello Vue'
        //     }
        // },
    data = vm._data = typeof data === 'function' //给_data赋值 判断data是不是函数 是函数执行 getData，不是则为data
      ? getData(data, vm)
      : data || {}; // 执行过getData方法之后 data = vm._data = {msg: 'Hello Vue'}
    if (!isPlainObject(data)) { // 是否是纯粹的对象 isPlainObject函数内容为：return _toString.call(obj) === '[object Object]'
      data = {}; // 不是对象 警告报错 所以data传值要么是一个函数 但是要返回值要为一个对象，要么data直接就是一个对象，除了这两个都报错
      warn(
        'data functions should return an object:\n' +
        'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
        vm
      );
    }
    // proxy data on instance
    var keys = Object.keys(data); // 获取data对象的键 这里 keys = ['msg']
    var props = vm.$options.props; // 获取props
    var methods = vm.$options.methods; // 获取methods
    var i = keys.length;
    while (i--) {// 循环
      var key = keys[i];
      {
        if (methods && hasOwn(methods, key)) { // 先不管methods
          warn(
            ("Method \"" + key + "\" has already been defined as a data property."),
            vm
          );
        }
      }
      if (props && hasOwn(props, key)) { // 先不管props
        warn(
          "The data property \"" + key + "\" is already declared as a prop. " +
          "Use prop default value instead.",
          vm
        );
      } else if (!isReserved(key)) { // 检查一个字符串是否以$或者_开头
        proxy(vm, "_data", key); // 不是以$或者_开头 ，将键值放入到_data 即 vm._data.msg = 'Hello Vue'
      }
    }
    // observe data
    observe(data, true /* asRootData */);
  }
```

```javascript
function getData (data, vm) {// 该方法核心为调用方法 返回函数值
    // #7573 disable dep collection when invoking data getters
    pushTarget(); // push进入一个数组， 暂且不理
    try {
      return data.call(vm, vm) // 即调用data函数 返回其中值，所以data一般要求写成一个函数 格式为： data() {return {}}
    } catch (e) {
      handleError(e, vm, "data()");
      return {}
    } finally {
      popTarget(); // 从数组移出 ， 暂且不理
    }
  }
```

```javascript
function observe (value, asRootData) {
    if (!isObject(value) || value instanceof VNode) { // 不是对象 或者 value的原型不在VNode的原型链上 直接返回
      return
    }
    var ob;
    if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) { // value有__ob__且value.__ob__的原型在Observer的原型链上
      ob = value.__ob__;
    } else if (
      shouldObserve &&
      !isServerRendering() &&
      (Array.isArray(value) || isPlainObject(value)) &&
      Object.isExtensible(value) &&
      !value._isVue // shouldObserve定义为true isServerRendering是否服务器渲染以及是否是数组是否是对象且Object.isExtensible(是否可扩展性)
    ) {
      ob = new Observer(value);
    }
    if (asRootData && ob) {
      ob.vmCount++;
    }
    return ob
  }
```

```javascript
var Observer = function Observer (value) {
    this.value = value; 
    this.dep = new Dep(); // Dep 消息管理器 粗略讲一下 Observer劫持数据 Watcher订阅信息，所以需要一个Dep消息管理器中转，连接Observer和Watcher
    this.vmCount = 0;
    def(value, '__ob__', this); // 把this赋值给value对象的__ob__属性 即 value.__ob__ = {dep:new Dep(),value,vaCount = 0}
    if (Array.isArray(value)) { //value是否是对象
      if (hasProto) {
        protoAugment(value, arrayMethods);
      } else {
        copyAugment(value, arrayMethods, arrayKeys);
      }
      this.observeArray(value);
    } else {
      this.walk(value);
    }
  };
```

```javascript
Observer.prototype.walk = function walk (obj) {
    var keys = Object.keys(obj); // 获取对象属性
    for (var i = 0; i < keys.length; i++) {
        defineReactive$$1(obj, keys[i]);
    }
};
```

```javascript
function defineReactive$$1 (
obj,
 key,
 val,
 customSetter,
 shallow
) {
    var dep = new Dep();

    var property = Object.getOwnPropertyDescriptor(obj, key); // 获得对象属性描述符
    //var o, d;
	//o = { get foo() { return 17; } };
    // d = Object.getOwnPropertyDescriptor(o, "foo");
    // d {
    //   configurable: true, // 可配置 为true可以被改变或者属性可被删除
    //   enumerable: true, // 可枚举(为true意思就是可以遍历)
    //   get: /*the getter function*/,
    //   set: undefined
    // }
    if (property && property.configurable === false) { // 如果属性不可配置 即返回不进行劫持 
        return
    }

    // cater for pre-defined getter/setters
    var getter = property && property.get;
    var setter = property && property.set;
    if ((!getter || setter) && arguments.length === 2) {// 如果getter为空或者setter不为空且defineReactive$$1传递的参数长度为2 赋值val
        val = obj[key];
    }

    var childOb = !shallow && observe(val); 
    // observe(val) 递归 为什么要递归解释一下 我们这里的data为{msg: "Hello Vue"}，但是如果为{msg: {test:'xxx'}}
    // 不止要劫持msg 还要劫持test，这就是我们为什么设置data里面，对象嵌套对象，里面的属性值都能双向绑定的原因。
    // 劫持对象中的每个属性，属性还是对象，递归继续劫持
    Object.defineProperty(obj, key, { // 开始使用Object.defineProperty劫持
        enumerable: true,
        configurable: true,
        get: function reactiveGetter () {
            var value = getter ? getter.call(obj) : val;// 如果getter不为空 调用getter
            if (Dep.target) {
                dep.depend();
                if (childOb) {
                    childOb.dep.depend();
                    if (Array.isArray(value)) {
                        dependArray(value);
                    }
                }
            }
            return value
        },
        set: function reactiveSetter (newVal) {
            var value = getter ? getter.call(obj) : val;
            /* eslint-disable no-self-compare */
            if (newVal === value || (newVal !== newVal && value !== value)) {
                return
            }
            /* eslint-enable no-self-compare */
            if (customSetter) {
                customSetter();
            }
            // #7981: for accessor properties without setter
            if (getter && !setter) { return }
            if (setter) {
                setter.call(obj, newVal);
            } else {
                val = newVal;
            }
            childOb = !shallow && observe(newVal);
            dep.notify();
        }
    });
}
```

