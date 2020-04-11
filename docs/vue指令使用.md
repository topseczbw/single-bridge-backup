# vue指令使用

使用vue两年了，指令用的非常少。

反思了一下原因，使用指令拼的是原生js功底，mvvm流行让开发者手动操作dom的次数越来也少。

这次特地啃一下这块的知识点，同时警示自己红宝书该翻一翻了。

## 什么是指令

> 指令是带有 v- 前缀的特殊属性，指令属性的值预期是单个JavaScript表达式。</br>

## 指令产生的原因

> 指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于dom。

指令的使命是用来操作dom，开发vue应用时，每当有dom操作，应该首先想到使用指令。

封装指令也是为了在遇到相同dom操作时，用简单的方式复用。

## 有哪些应用场景

- 场景1：click-outside场景，当点击非指定元素外的其他元素时，指定关闭/收起下拉框，遮罩的操作

```html
<div v-click-outside='closeFn'>
</div>
```

```js
export default {
  name: 'ClickOutside',
    bind(el, binding) {
      function documentHandler(e) {
        if (el.contains(e.target)) {
          return false
        }
        if (binding.expression) {
          binding.value(e)
        }
      }
      el.__vueClickOutside__ = documentHandler
      document.addEventListener('click', documentHandler)
    },
    unbind(el) {
      document.removeEventListener('click', el.__vueClickOutside__)
      delete el.__vueClickOutside__
    }
  }
```

- 场景2：如在钉钉端应用中，点击图片元素，实现全屏查看。

  我们可以给放置图片的容器元素绑定指令 `<div class="box" v-ImgPreviewDirective>`，由于点击事件会冒泡到父级元素，所以在指令具体实现中，监听父级元素（指定绑定元素）的点击事件，判断 `target` 是否是图片元素，如果是则触发执行器函数。

  执行器函数中使用适配器模式，判断当前执行的环境是钉钉还是pc端，执行不同的处理逻辑

  这样一来，就将全屏显示图片操作的所有细节逻辑都封装在了一个指令中，轻松地实现复用。

## 个人理解

### 参数

>除了 el 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 dataset 来进行。

既然为了操作dom，在指令中一定可以获取到dom元素、参数（常指dom属性）、修饰符、绑定值（用户自定义参数）

```js
el: 即指定绑定的dom元素

binding: {
  name: 指令名称
  value: 传入的用户自定义参数
  arg: 如v-bind:href 中的href 指相关联的dom属性
  modifiers: 修饰符相关
}

vNode：指定所在的vue组件的虚拟节点，通过vNode的context属性可以获取到指令所在的vue实例
```

一般操作指令时，希望传入更多的用户自定义参数

```html
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
```

如上，即可通过 `binding.value.color` 获取到较多的参数

### 动态参数

很少用到，如 `v-style:[attr]="200"` 对于传入的指令的参数（dom属性）不固定，可能是高，也可能是宽的时候

### 钩子函数

bind: 只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置

inserted： 被绑定元素插入父节点时调用（仅保证父节点存在，但不一定插入文当中），至此可以获取到el对象

update: `所在组件的vNode更新时调用, 但是可能发生在其子 VNode 更新之前。` 注意！即当视图改变，钩子就会被调用，这个钩子一般情况下会被多次调用

componentUpdated: 指令所在组件的 VNode 及其子 VNode 全部更新后调用。

unbind: 只调用一次，指令与元素解绑时调用。
