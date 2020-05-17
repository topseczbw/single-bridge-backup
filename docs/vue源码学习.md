# vue源码学习
<!-- TOC -->

- [MVVM](#mvvm)
- [new Vue的时候做了些什么](#new-vue的时候做了些什么)
- [数据劫持](#数据劫持)
  - [什么是数据劫持](#什么是数据劫持)
  - [如何实现数据劫持](#如何实现数据劫持)
  - [为了取值更方便，进行代理](#为了取值更方便进行代理)
- [数组劫持](#数组劫持)
  - [数组劫持背景](#数组劫持背景)
  - [对初始化data中数组对象每一项进行监测](#对初始化data中数组对象每一项进行监测)
  - [对后续数组的新增项进行监测](#对后续数组的新增项进行监测)
- [模板编译](#模板编译)
  - [什么是模板编译](#什么是模板编译)
  - [vue实例渲染顺序优先级](#vue实例渲染顺序优先级)
  - [AST语法树对象和虚拟dom对象的区别](#ast语法树对象和虚拟dom对象的区别)
  - [模板编译操作具体如何实现](#模板编译操作具体如何实现)
- [初始化渲染](#初始化渲染)
  - [_render方法](#_render方法)
  - [具体过程](#具体过程)
  - [数据修改，视图需要更新时做了点什么](#数据修改视图需要更新时做了点什么)
  - [每一次都需要生成AST语法树吗](#每一次都需要生成ast语法树吗)
- [生命周期的合并策略和执行](#生命周期的合并策略和执行)
  - [mixin的合并策略](#mixin的合并策略)
  - [初始化全局API/静态方法](#初始化全局api静态方法)
  - [全局Vue.mixin的实现](#全局vuemixin的实现)
  - [callHook（vm, hook）](#callhookvm-hook)
- [对象的依赖收集](#对象的依赖收集)
  - [解决watcher重复存放的问题](#解决watcher重复存放的问题)
- [数组的依赖收集](#数组的依赖收集)
  - [数组什么时候notify依赖更新](#数组什么时候notify依赖更新)
  - [数组什么时候进行依赖收集](#数组什么时候进行依赖收集)
  - [当data选项中存在二维数组或者三维数组的情况呢](#当data选项中存在二维数组或者三维数组的情况呢)

<!-- /TOC -->
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

### 为了取值更方便，进行代理

vm.name 代理到 vm._data.name，这里代理一层即可，不需要递归

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

当传入el选项时，使用outerHTML属性可获取包含自身在内的html字符串

### AST语法树对象和虚拟dom对象的区别

AST语法树是用对象来描述原生html语法。AST语法树并不关心vue相关的属性，比如v-model属性也会被AST视为html标签普通的属性。并且AST对象存在nodeType（1元素节点 3文本节点）标识类型。

虚拟dom是用对象来描述真实dom节点。虚拟dom与vue相关，可以添加自定义属性。如v-model这样的指令就会被解析为input事件和value属性，更关心vue这个层面的东西

### 模板编译操作具体如何实现

解析html字符串：`将字符串 => AST语法树 => render函数字符串 => render函数对象`

在获取到 `html模板字符串` 后，使用正则进行循环匹配开始标签、属性、结束标签解析html字符串 `parseHTML` ，在每次匹配后对html字符串调用 `advance` 方法，截断字符串

在这个过程中可以获取到每个标签 及其属性，接下来会根据这些数据创建 `AST语法树对象`

具体过程：

1. 当匹配到开始标签时，使用 `createAstElement` 方法创建AST语法树对象，并为此对象添加 `attrs` 属性。如果此时没有root对象，则该对象为根对象。将此对象标记为 `currentParent`。
2. 当匹配到文本标签时，过滤删除空白字符后，将创建文本元素对象（type = 3）并将其放入 `currentParent` 的children中
3. 当匹配到结束标签的时候。

   会遇到一个问题：在这个过程中如何知道标签是否正常结束了？

   在此使用栈结构辅助判断，每当匹配到一个开始标签，就将其放入栈中，每当匹配到一个结束标签，就拿栈顶的标签作比较，如果相同就移除栈顶的标签，同时判断 stack[length - 1] 元素是否存在，如果存在，则说明stack[length - 1] 元素是当前标签的父标签，则为父标签添加children属性。

   为什么只有在匹配到结束标签时，才会增加父子级关系？

   因为在匹配开始标签的时候，事先并不知道子标签是啥。栈结构是顺序列表结构，没有父子级关系，只有在匹配到关闭标签的时候，可以通过上一个索引知道父标签是谁，如果没有找到，说明没有父标签，是root

到这里，成功的将html字符串转化为AST语法树对象，下面需要做的是将`AST语法树对象转成render函数`。用到的核心原理：`字符串拼接/模板引擎`

generator方法传入ast语法树，输出render函数字符串（js语法）。

1. 处理属性：过程中对style属性需要做特殊处理
2. 处理孩子

   每个孩子用逗号分隔

3. 处理文本

   需要对 `{{}}` 进行特殊处理：在外面包裹一层 `_s()` 标识这是一个标量，不是字符串。如：`_v("hello" + _s(name))` **这段字符串最终会在vm（当前vue实例）为this的环境下执行，即会获取vm上的属性，触发之前做的数据劫持相关方法**

   在这里会用到全局的正则匹配，注意只要是全局匹配，就需要每次讲lastIndex变成0

经过以上处理可得到一个render函数字符串，下面将字符串转化为真正的render函数，核心原理同模板引擎。所有的模板引擎原理都是使用 `new Function将字符串生成函数对象 + 并在函数中使用with关键字，让字符串中的变量拥有上下文`

最终得到render函数

---

## 初始化渲染

终于要渲染页面了

得到render函数之后，下面应该做的就是渲染当前组件、挂载这个组件到指定的dom上。`mountComponent`

1. updateComponent方法：无论是初次渲染还是更新都会调用此方法
2. 渲染watcher: 每个组件都有一个渲染watcher，用来渲染页面
3. vm._render方法：通过解析的render函数，生成虚拟dom对象
4. vm._update方法：通过虚拟dom，创建真实dom

### _render方法

1. _c:创建元素的虚拟节点
2. _v:创建文本的虚拟节点
3. _s:JSON.stringify

### 具体过程

1. 调用 `patch（vm.$el, vnode)` 方法，通过虚拟dom递归创建真实节点，在用新节点**替换**掉老节点，这里详细操作是，获取新节点后先插入到目标dom后面，再将目标从父元素中移除，是为了保证在准确地同一位置替换
2. 由于patch方法在数据更新时也需要调用。首次渲染不同与后续更新的地方是在传入的第一个参数，首次传入的是真实dom元素，更新传入的是旧虚拟dom
3. 在递归调用 `createElm` 方法中将真实dom元素映射到虚拟dom上，即vnode.el属性是真实dom，方便后续操作

### 数据修改，视图需要更新时做了点什么

在初次渲染的时候，我们已经通过编译/不编译的手段将vm的render方法绑定在vm上。等下次数据更新时，可以直接获取实例上的_render方法，用新的数据生成新的虚拟dom，与旧的虚拟dom做diff对比(patch)，最小化的操作dom，实现视图更新，此处通过nodeType属性判断是否真实节点

### 每一次都需要生成AST语法树吗

模板不变 ast不变

在开发时，由于需要改动模板做调试，所以会经常更新ast语法树。但是真正生产环境时，由于模板已经稳定不变，所以只有在第一次模板解析时会生成，只有数据变动，只是调用render方法，新旧虚拟dom对比，改动

## 生命周期的合并策略和执行

整个渲染流程熟悉了之后，下面我们会给整个流程添加一些钩子，方便用户在特定的阶段执行自定义逻辑

### mixin的合并策略

mixin核心方法 `mergeOptions`，规则即对象合并，同名的key后者会覆盖前者。但是遇到声明周期钩子时，需要使用数组的形式保存。

### 初始化全局API/静态方法

调用`initGlobalApi`方法，接收参数Vue

### 全局Vue.mixin的实现

实际上是在Vue构造函数上有一个静态属性 option 存放着全局的混合。然后在每次初始化vue实例时，同样调用`mergeOptions`将全局的混合与当前vm的混合合并。

```js
vm.$options = mergeOptions(vm.constructor.options, options)
```

这里使用`vm.constructor.options`而没使用Vue，是因为考虑到后续可能出现 a extend Vue 使用组件继承的方式，new a（）去创建组件，所以这里全局的mixin不一样全都放在Vue的静态属性options上，还有可能放在子类上

### callHook（vm, hook）

`callHook（vm, 'mounted')`  `callHook（vm, 'created')` 等函数在合适的地方调用，内部会从vm.$options中筛选出对应名称的钩子函数列表，循环执行

## 对象的依赖收集

初次渲染搞定了，钩子函数也有了。

下面我们希望修改数据后，视图会自动更新，而不需要用户再手动调用 vm._update(_render()) 方法去更新dom。

所以就需要进行依赖收集操作。

每个属性都有一个dep依赖对象，dep存放着该属性相关的所有watcher，**每个属性只有一个dep**

watcher种类：渲染watcher、computed计算函数、watch回调函数

**watcher可以理解为，某个属性改变时，其相关联的操作的执行函数**，如：需要更新视图或者更新computed或者调用watch监听的回调

以渲染watcher为例

```js
// 此处是执行mountedComponent时，初此渲染视图的方法
pushTarget(this)
this.getter.call(this.vm)
popTarget()
```

```js
Object.defineProperty(data, key, {
    get() {
      // 取数据之前 已经把 watcher 放到了target上 【source/vue/observe/watcher.js:37】
      // todo 注意：同一个属性可能会在模板中被多次取值，有可能会在dep中注册很多相同的watcher，我们希望watcher不能重复，如果重复了就会造成更新时，多次渲染
      if (Dep.target) {
        // watcher 和 Dep 互相依赖
        dep.depend() // 想让dep中 可以存watcher 还希望让这个watcher中存放dep 实现多对多关系


        // 如在使用 vm.list 时
        // 在这里不用担心对象会重新收集  因为在【source/vue/observe/watcher.js:46】方法中会判断dep唯一标识
        if(childOb) {
          childOb.dep.depend()

          // 递归收集儿子的依赖
          dependArray(value)
        }
      }
      console.log('获取数据，【渲染dom，更新试图】')
      return value
    },
    set(newValue) {
      if (newValue === value) return
      console.log('设置数据  设置vm属性')
      observe(newValue)
      value = newValue

      // 执行 该属性 订阅过的 watcher
      dep.notify()
    }
  })
```

先将渲染watcher放在Dep.target上。模板解析完毕后，执行render函数生成真实dom时，会触发对vm实例属性的get取值操作，此时该渲染watcher会被该属性的dep收集，在下次修改该属性时，触发set方法notify通知该属性关联的所有watcher，重新执行。达到更新视图的目的。

每个属性都对应着自己的watcher列表

### 解决watcher重复存放的问题

如果模板中对于同一个属性，存在多次取值的场景如：

```html
<div>{{name}} {{name}} {{name}}</div>
```

这种情况下，每次取name属性，dep中就会增加一个渲染watcher，结果导致name属性的dep中存在三个相同的渲染watcher。同一个属性在模板中被多次取值，有可能会在dep中注册很多相同的watcher，watcher重复了就会造成更新时，多次渲染

要解决上述问题，需要先搞清楚 watcher 和 dep 的关系是什么？

先前的关系是，每个属性的唯一dep中存放着多个watcher，一对多，相当于属性记录了自己更新时，需要执行那些函数，触发那些方法，更新哪部分视图。但是watcher并不知道自己被哪些属性所使用。

故将watcher 和 dep 改为多对多，watcher中记录去重后的dep，dep中也记录着唯一的watcher。每次触发属性的get方法时，调用 `dep.depend() => Dep.target.addDep(this)` 方法将dep存放到watcher上，在watcher中使用Set集合结构去重后，将dep存放起来后，再将watcher记录到dep上。由此保证每个属性的dep中 不会出现重复的watcher

渲染watcher单个组件只有一个，但是如果有多个组件即多个vm实例，则会有多个渲染watcher

## 数组的依赖收集

### 数组什么时候notify依赖更新

设想一下，对于数组什么时候会触发update，应该是在调用数组的方法如push时，我们希望视图可以更新，所以我们会在push的时候，通知当前数组的所有依赖watcher，调用dep的notify方法，更新视图或者computed，那么问题来了，但是我们在什么时候收集依赖呢？

### 数组什么时候进行依赖收集

回想收集对象的依赖是在defineReactive函数中，在defineReactive为每个属性定义set和get时，利用闭包，同时为每个属性定义一个dep，在get时，将依赖watcher们收集到dep中，在set时依赖可以获取到闭包中的watcher，调用notify通知视图更新

对于数组，如vm.list.push()操作时，我们希望视图自动更新，我们已经知道在push时notify。我们可以在前半段操作 vm.list 中处理，在defineReactive中vm.list 的value由于是数组，在模板解析中，如果用到list属性，会递归对value也进行监测， 会observer（value）。所有我们可以在defineReactive中获取到监测数组返回的childObj（属性代表的observer实例），这样就可以在defineReactive中获取到数组的dep实例，进行依赖收集。

对象的dep实例在 defineReactive 的闭包里存着

数组的dep实例放在Observer实例上，在数组被当做对象的属性 value observer监测 时，返回值可以获取到Observer实例，即可以获取到数组的dep依赖

### 当data选项中存在二维数组或者三维数组的情况呢

当模板中存在 list[0][1].name时呢 ，如何收集依赖

在 defineReactive 中对数组进行依赖收集后，同时判断数组的item是否还是数组，递归进行依赖收集

```js
/*
 * @Author: zbw
 * @Date: 2020-03-18 21:26
 */

import {observe} from "./index";
import {arrayMethods, observerArray} from "./array";
import { Dep } from './dep'

export function defineReactive(data, key, value) {

  // 如果属性是对象，递归观察
  // todo childOb 专门服务于数组 是数组的那个dep
  let childOb = observe(value)
  // todo 注意：这是一个闭包  由于get、set方法可以获取到 defineReactive 方法作用域中的变量，因此在外部修改属性时， dep 可以一直存活，一直被访问到
  // 这个dep是给对象用的
  let dep = new Dep()
  Object.defineProperty(data, key, {
    get() {
      // 取数据之前 已经把 watcher 放到了target上 【source/vue/observe/watcher.js:37】
      // todo 注意：同一个属性可能会在模板中被多次取值，有可能会在dep中注册很多相同的watcher，我们希望watcher不能重复，如果重复了就会造成更新时，多次渲染
      if (Dep.target) {
        // watcher 和 Dep 互相依赖
        dep.depend() // 想让dep中 可以存watcher 还希望让这个watcher中存放dep 实现多对多关系


        // 如在使用 vm.list 时
        // 在这里不用担心对象会重新收集  因为在【source/vue/observe/watcher.js:46】方法中会判断dep唯一标识
        if(childOb) {
          childOb.dep.depend()

          // 递归收集儿子的依赖
          dependArray(value)
        }
      }
      console.log('获取数据，【渲染dom，更新试图】')
      return value
    },
    set(newValue) {
      if (newValue === value) return
      console.log('设置数据  设置vm属性')
      observe(newValue)
      value = newValue

      // 执行 该属性 订阅过的 watcher
      dep.notify()
    }
  })
}

/**
 * 递归收集数组中的依赖
 * @param value
 */
export function dependArray(value) {
  for (let i = 0; i < value.length; i++) {
    // 有可能也是一个数组
    let currentItem = value[i]
    currentItem.__ob__ && currentItem.__ob__.dep.depend()

    if (Array.isArray(currentItem)) {
      // 不停的收集数组中的依赖关系
      dependArray(currentItem)
    }
  }
}

class Observer {
  constructor(data) {

    // 这个dep专门为数组使用
    // 对象的依赖 在【source/vue/observe/observer.js:16】闭包中收集
    this.dep = new Dep()
    // 为从data开始的每个对象、数组
    // 为每个属性都增加一个__ob__属性，返回的就是当前的Observer实例
    // 这样做是为了让数组属性可以获取到Observer实例
    Object.defineProperty(data, '__ob__', {
      get: () => this
    })
    if(Array.isArray(data)) {
      // 对现有数组的每一项进行观察
      observerArray(data)

      // 对未来某一刻  数组可能会有插入操作  那么对插入的每项也进行观察
      // todo 如果是数组  对新增项、原有项是对象的进行观察
      // todo 通过改变data上数组对象的原型链  使在vue实例上声明的数组属性被劫持 只有传入的数据需要被劫持
      data.__proto__ = arrayMethods
    } else {
      // 如果是对象的话
      this.walk(data)
    }
  }

  walk(data) {
    let keys = Object.keys(data)
    for (let i = 0; i < keys.length; i++) {
      let key = keys[i]
      let value = data[keys[i]]

      defineReactive(data, key, value)
    }
  }
}

export default Observer
```
