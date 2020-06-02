# 读vue-router源码

分析源码前不妨先问自己几个问题，有目的的看源码，以终为始

1. 回想一下 `vue-router` 在项目中是如何使用的，分几步
2. 如何实现在不刷新页面的前提下改变url
3. 配置项中传入的mode作用
4. $router 和 $route 是什么，从何而来
5. router-link 和 router-view 是什么，从何而来
6. 如何实现根据不同url渲染不同组件

## vue-router 在项目中如何使用

1. 首先 `npm i vue-router` 安装依赖
2. VueRouter 是一个构造函数，通过 new 并传入配置项包括 `mode` `routes` 模式和path映射，初始化router实例，生成实例后。导出并且在根组件（main.js）中，做为自定义选项，注入根组件
3. 使用 `Vue.use(VueRouter)` 以插件的形式，安装router

## 如何实现在不刷新页面的前提下改变url

两种方式如下：

1. `hash` 锚点的方式

    ```html
    <a href="#/home">首页</a>
    <a href="#/about">关于</a>
    <div id="html"></div>
    ```

    ```js
    // 监听 window 的 `load` 事件，初始化渲染组件
    window.addEventListener('load', () => {
      html.innerHTML = location.hash.slice(1)
    })
    // 监听 window 的 `hashchange` 事件，动态渲染不同组件
    window.addEventListener('hashchange', () => {
      html.innerHTML = location.hash.slice(1)
    })
    ```

2. H5 History Api 方式

   注意在页面刷新的时候，如当前url为 `www.xxx.com/home`，服务器上没有home文件夹，所以找不到对应的html资源，会出现404，所以在使用history模式时，需要跟后台配置支持。

   在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的入口页面。

   ```html
    <a onclick="go('/home')">首页</a>
    <a onclick="go('/about')">关于</a>
    <div id="html"></div>
   ```

   ```js
   // 使用history的pushState方法，实现改变url且不刷新页面，动态渲染不同组件
   function go(pathName) {
      history.pushState({}, null, pathName)

      html.innerHTML = pathName
    }
   // 监听浏览器前进后退事件，动态渲染不同组件
    window.addEventListener('popstate', () => {
      go(location.pathname)
    })

   ```

## 配置项中传入的mode作用

   mode选项即用来配置当前vue应用使用 hash模式 还是 history模式

## $router 和 $route 是什么，从何而来

   `$router` 标识全局路由器对象，即 `new VueRouter` 生成的实例，可以通过调用 `$router` 实例上的方法，实现路由跳转

   `$route` 是标签与当前url匹配的路由对象，可以用来获取当前url上的query参数

   `$router` 和 `$route` 都是vue实例上的属性，都是在 `Vue.use(VueRouter)` 时，通过 `Vue.mixin` 方法，配合 `beforeCreate` 钩子函数，为每个vue实例增加的属性

## router-link 和 router-view 是什么，从何而来

`router-link` 组件支持用户在具有路由功能的应用中 (点击) 导航。

与写死的 `<a href="...">` 对比，优势：

- 无论是hash还是history模式，表现行为一致
- 在 HTML5 history 模式下，router-link 会守卫点击事件，让浏览器不再重新加载页面。
- 当你在 HTML5 history 模式下使用 base 选项之后，所有的 to 属性都不需要写 (基路径) 了。

`<router-view>` 组件是一个 functional 组件，渲染路径匹配到的视图组件。

`router-link` 和 `router-view` 也是在 `Vue.use(VueRouter)` 时，通过 `Vue.component` 方法注册的全局组件

这里组件渲染其实使用的是render函数

```js
Vue.component("router-link", {
  props: {
    to: String,
    tag: String
  },
  methods: {
    handleClick() {
      alert(1);
    }
  },
  render() {
    // return h("a", {}, "首页");
    let mode = this._self.$router.mode;
    let tag = this.tag;
    return (
      <tag
        on-click={this.handleClick}
        href={mode === "hash" ? `#${this.to}` : `${this.to}`}
      >
        {this.$slots.default}
      </tag>
    );
  }
});

Vue.component("router-view", {
  render(h) {
    let current = this._self.$router.history.current;
    let routeMap = this._self.$router.routeMap;
    return h(routeMap[current]);
  }
});
```

## 如何实现根据不同url渲染不同组件

   在 `new VueRouter` 的时候，会在构造函数中，根据传入的mode模式，启动不同的对url的事件监听方法

   并在 `$router` 实例中增加属性 `history.current` 存储当前页面路由对象

   当用户触发url改变操作时，如调用 `push` `go` `back` 的方法时，会改变 `history.current`

   这里发现，虽然history对象修改了，但是视图仍然无法刷新

   此时，使用 `Vue.util.defineReactive` 方法，给当前vue实例，随机绑定一个响应式属性，值为 `this._router.history` 对象，达到当`this._router.history` 发生变化时 ，会更新视图
