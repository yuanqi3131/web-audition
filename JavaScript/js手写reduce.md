#### JS手写reduce



```javascript
<script>
    Array.prototype.myReduce = function (fn, init) {
        var total = init >= 0 ? init : this[0];
   		// 为什么不能写成 var total = init || this[0](init为0的时候也执行this[0])
        for (let i = 0; i < this.length; i++) {
            total = fn(total, this[i], i, this)
        }
        return total;
    }

    let data = [1, 2, 3, 4, 5, 6, 7, 8]

    function add(a, b) {
        return a + b
    }
    console.log(data.myReduce(add, 0)) // 36

</script>
```

