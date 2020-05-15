# vue源码学习

## MVVM

核心原理：数据变化可以驱动视图变化

---

## new Vue的时候做了些什么

`new Vue()` 时，我们会传入一些配置，进行初始化vue实例操作，一个Vue实例即一个组件

> 源码中使用es5 function构造函数的形式声明Vue，而没有使用es6 class的形式去声明，是因为这样做可以将Vue类的原型方法拆分到不同的文件中，使代码更加清晰。
>
> 具体实现是通过将Vue构造函数作为参数，传入不同文件暴露出来的mixin方法，在mixin方法中，对Vue构造函数的原型对象进行扩展

---

## 数据劫持

### 什么是数据劫持

用户改变了数据，我们希望得到通知，然后去刷新页面，更新视图

> 在用户操作数据的时候，视图会进行更新，原因是Vue劫持了用户在初始化Vue实例时，选项中传入数据data，在data修改时，Vue会使用新的数据，再次渲染视图

### 如何实现数据劫持

vue的数据来源有很多种：props、methods、data、computed、watch等

再此只分析new Vue时如何处理data选项 `initData` 方法

```js
data = vm._data = typeof data === 'function' ? data.call(vm) : data || {}
```

> 传入的data可能是对象，也有可能会使用到props初始化属性，所以当data选项是方法时，使用data.call方法，将this指向当前vue实例简称 `vm`
>
> 至此vm._data即vm上的一个实例属性对象，所以如果data选项最初传入一个对象而不是方法时，多个vm实例会共享这个对象

然后使用 `observe` 方法对 `vm._data` 对象进行监测，原理是使用 `Object.defineProperty(data, key, value)` 方法对vm._data对象设置set和get方法，进行 `递归` 监测（实际在监测的过程中会进行依赖收集）

> 由于是对data对象进行递归劫持，所以当数据层次过多时，性能会差

注意：

1. 当data对象的root层属性也是对象时，需要进行递归监测，即对value也需要进行observe
2. 用户会发生重新赋值的操作如：`this._data.zbw = {name: 'zbw'}`,此时我们也希望新赋值的对象属性也被监测，所以在set方法中，需要对新的value值也进行监测observe

---

## 数组劫持

### 数组劫持背景

前面做了对象劫持的操作。当data中root层属性是数组时，需要对声明的数组类型的对象的每一项也做监测。同时考虑我们很少通过数组的索引访问item对象，而是经常会对数组进行增删，删除就直接删了就好，不用考虑劫持问题，但是对于有可能造成当前数组新增项的操作，需要新增加的item项进行劫持。

### 对初始化data中数组对象每一项进行监测

在 Observer 类中，增加判断，如果value是数组的话，调用 `observerArray` 方法循环数组中的每一项，调用 `defineReactive` 方法

### 对后续数组的新增项进行监测

原理：拦截原生数组的部分方法，对新增项进行劫持

由 `Array.prototype` 为原型对象新创建一个中间层对象 `arrayMethods` ，

```js
let oldArrayProtoMethods = Array.prototype

export let arrayMethods = Object.create(oldArrayProtoMethods)
```

让当前 `value`（只是当前vue实例中_data对象中的数组对象，不会影响当前vue实例外的数组）的原型对象指向 `arrayMethods`

借助原型链规则和AOP面向切面编程的思想，在调用vm上的数组对象的方法（修改原数组对象的方法如：push、splice、unshift）时，会先走 `arrayMethods` 对象的push方法（我们自定义的方法）。

在自定义方法中，先调用 `oldArrayProtoMethods` 的对应方法，保证改变原数组数据。而后获取到新增项列表，对该列表进行循环监测。

---

## 模板编译

至此，数据劫持完了，我们需要把数据渲染到页面上。

如果用户传入了el属性，需要将页面渲染出来

### 什么是模板编译

模板编译就是先将带有模板语法的`html字符串`编译成`AST语法树对象`，然后再利用`AST语法树对象`生成`render函数字符串`，进而生成`render函数`

### vue实例渲染顺序优先级

render选项 > template选项 > el选项

不管你是写 template选项、el选项、.vue单文件组件（由vue-loader去编译），最终都会转化成render函数去渲染页面

当选项中没有render函数时，并且存在template或者el选项时，说明需要模板编译

### AST语法树对象和虚拟dom对象的区别

AST语法树是用对象来描述原生html语法

虚拟dom是用对象来描述真实dom节点

### 模板编译操作具体如何实现

在获取到`html模板字符串`后，使用正则进行循环截取

### 模板编译注意

1. outerHTML属性可获取包含自身在内的html字符串

---
