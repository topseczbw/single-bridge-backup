# vue源码解读

<!-- TOC -->

- [vue源码解读](#vue%e6%ba%90%e7%a0%81%e8%a7%a3%e8%af%bb)
  - [准备工作](#%e5%87%86%e5%a4%87%e5%b7%a5%e4%bd%9c)
    - [flow](#flow)
    - [目录结构](#%e7%9b%ae%e5%bd%95%e7%bb%93%e6%9e%84)
    - [vue源码构建](#vue%e6%ba%90%e7%a0%81%e6%9e%84%e5%bb%ba)
    - [从入口开始](#%e4%bb%8e%e5%85%a5%e5%8f%a3%e5%bc%80%e5%a7%8b)
  - [数据驱动](#%e6%95%b0%e6%8d%ae%e9%a9%b1%e5%8a%a8)
    - [从new Vue开始看看都发生了哪些事情](#%e4%bb%8enew-vue%e5%bc%80%e5%a7%8b%e7%9c%8b%e7%9c%8b%e9%83%bd%e5%8f%91%e7%94%9f%e4%ba%86%e5%93%aa%e4%ba%9b%e4%ba%8b%e6%83%85)
    - [vue实例挂载的实现(重要)](#vue%e5%ae%9e%e4%be%8b%e6%8c%82%e8%bd%bd%e7%9a%84%e5%ae%9e%e7%8e%b0%e9%87%8d%e8%a6%81)
    - [render](#render)
    - [virtual DOM](#virtual-dom)
    - [createElement(tag, ?dataOption, children)](#createelementtag-dataoption-children)

<!-- /TOC -->

## 准备工作

### flow

1. 在编译期间尽早发现bug
2. 类型检查的两种方式：类型推断、类型注释
3. npm i -g flow-bin
4. flow init
5. 根目录 flow 文件夹 定义了 vue中使用的类型

### 目录结构

1. compiler 模板编译相关
2. core vue核心

   组件
   全局api
   实例
   响应式
   工具函数
   虚拟dom
3. platforms 平台相关的  web 和 weex(native)
4. server 服务器端渲染相关
5. sfc 将.vue单文件组件编译成js对象
6. shared  公用辅助函数 和 常量

### vue源码构建

1. 基于rollup构建  更轻量  更友好  更适合库
2. main 就是我们这个npm包的入口  就是我们import vue的入口
3. module 和 main类似 在webpack 2以上 会以module为入口
4. npm script  定义脚本  每个脚本都是一个任务
5. 构建相关  build  build:ssr  build:weex
6. dist目录下 有很多版本的vuejs
7. 使用脚手架构建项目是，会问我们是使用runtimeOnly 版本还是 runtime-with-compiler版本
8. runtimeOnly 同样借助 webpack 和 vue-loader  将我们写的各种形式的模板编译成 render函数 ，是在运行前编译， 在运行时是不会编译的，是离线时做的  单文件组件、 外链模板
9. runtime-with-compiler 可以在代码中写template模块选项时，在运行时动态编译 ,此时需要compiler版本
10. vue2 中所有的渲染都是生成render函数
11. 开发阶段 推荐使用runtimeOnly版本  省去运行时编译时间
12. 为了分析vue的编译过程，学习带compiler版本的js

### 从入口开始

入口在哪里？？？ entry-runtime-with-compiler

1. 讲讲从import vue都做了哪些事情
2. this instanceof Vue
3. 为什么要是使用fn实现vue 不使用class，为了隔离原型方法到各个文件， class的话就不弄了
4. import vue 追到源头    执行了 initGlobalAPI 定义了全局静态方法 Vue.set 等 实现了一个使用方法创建的vue类

## 数据驱动

dom变成了数据的映射，所有的逻辑都是数据的修改，使用简洁的模板语法，将数据渲染成dom

```html
<div id='app'>
  {{message}}
</div>
```

```js
var app = new Vue ({
  el: '#app',
  data: {
    message: 'zbw'
  }
})
```

只分析，数据是如何映射到dom的，传入了一个js对象，如何生成在dom上的

有目的的看源码，代码就像一棵树。
用到哪些东西，就看哪块

### 从new Vue开始看看都发生了哪些事情

1. 为什么mounted里面可以访问到this.message
2. _initMixin  _initState  initData  proxy

### vue实例挂载的实现(重要)

1. const vnode = vm._render()
2. vm._update(vnode, hydrating)
3. updateComponent方法

### render

1. 差值 和 手写render函数的区别  差值会先显示， 等new vue之后，在渲染， 而render函数会直接在new vue时渲染
2. 手写完render函数后，就不会再走模板的方法
3. 写了el  和 render   render生成的html 会替换掉 挂载所在的el

---

### virtual DOM

1. 浏览器的dom是非常昂贵的
2. 简单打印出div元素的所有属性  频繁操作dom
3. 用一个原生js对象去描述一个dom节点
4. VNode实际比真实dom要小很多
5. 虚拟dom参考snabbdom的实现  是vuejs的基础，有实现了很多拓展
6. 标签名  数据  子节点  键值
7. 由于vnode只是定义dom并渲染，并不包括操作dom的各个方法，所以比较轻量简单
8. 创建  diff  patch
9. vue 使用 createElement方法 创建   （实例上的render函数）

### createElement(tag, ?dataOption, children)

用来生成VNode，VNode Tree 完美映射到dom tree

1. render方法  最后会调用option.render函数
2. 模板编译render函数     手写的render函数
3. 定义在 vdom create-element.js
4. vnode一共有几种类型
5. 文本vnode
6. 拿到了vnode，如何生成一个真实的dom
7. _update方法 生成真实的dom
8. 首次渲染时、改变数据时
9. _patch
10. 节点替换 创建一个新的节点  把旧的节点替换了
