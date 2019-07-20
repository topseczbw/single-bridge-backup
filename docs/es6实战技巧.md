# es6实战技巧

## 数组

1. 使用 `includes` 代替 `indexOf`

   原因：不容易出错，很容易错写成  `arr.indexOf(xxxx) > 0` ， 正确应该是 `arr.indexOf(xxxx) > -1` ，归咎根本原因是后者不够语义化。

2. `const list = new Array(3)` 生成长度为3的 `empty` 元素list，这个list中的元素，是不被 `map` 方法受理的，`map` 可以识别 `undefined` 但不能识别空元素。可以使用 `Array.of(list)` 或 `[...list]` 将空元素转为 `undefined` 。

## 对象

1. 在vue中，使用 `Object.assign` 合并对象时，虽然最后的数据合并成功了，对象中属性该有的也都有了，但是vue的响应式数据监听 `setter` `getter` 只会采用base对象的，即方法的第一个参数。其他对象的属性   `setter` `getter` 会被忽略。这在一些特定应用场景中会造成问题。

   解决方案：

   ```js
   data() {
       return {
         zbw: {
           name: 'zbw'
         },
         ls: {
           name: 'ls',
           age: '18'
         },
         xt: {
           sex: '男'
         },
         wData: null
       }
    },
     mounted() {
       // this.zbw有监听
       console.log(this.zbw)
       // baseObj无监听
       let baseObj = {}
       Object.assign(baseObj, this.zbw, this.ls, this.xt)
       // baseObj无监听
       console.log(baseObj)
       // this.wData 有监听
       // baseObj 有监听
    		// 由于 wData 是 vue 中 一个数据对象，虽然 baseObj 无监听，但是赋值给vue中的数据以后，就有了监听。
       // 这个操作在 data 选项中给wData赋予了初始值
       // 此时属性全部合并完成，并且也成为了 vue 中的一个可监听的对象。
       this.wData = baseObj
     }
   ```

   

2. 213

3. xxxxxx

## 函数

1. 箭头函数的 `this` 指向

   箭头函数的 `this` 在函数创建时就已经决定了。

   箭头函数本身没有 `this` , **this 关键字指向的是它当前周围作用域**，简单来说是**包含箭头函数的常规函数**，如果没有常规函数的话就是**全局对象**，**但注意！对象不算常规函数*，箭头函数作为对象的属性时，寻找 `this` 时应该继续向上找。**

2. 函数参数默认值

   使用函数参数默认值一定要注意：**当且仅当传入值为 `undefined` 时，默认值生效，即当传入值为 `null` 时，默认值也是不会生效的。**这很容易让函数中的逻辑代码报错。

3. 

## 解构赋值

1. 