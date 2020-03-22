# 解读package.json

<!-- TOC -->

- [browserslist配置项](#browserslist配置项)
  - [是什么](#是什么)
  - [为什么会存在](#为什么会存在)
  - [有哪些应用场景](#有哪些应用场景)

<!-- /TOC -->

## browserslist配置项

### 是什么

browserslist是package.json包描述文件中的一个配置项，用于配置浏览器类型

```js
"browserslist": [
    "> 1%", // 表示包含所有使用率 > 1% 的浏览器
    "last 2 versions", // 表示包含浏览器最新的两个版本
    "not ie <= 8" // 表示不包含 ie8 及以下版本
]
```

### 为什么会存在

主要作用是用于在不同的前端工具之间共享目标浏览器和 Node.js 的版本

比如像 autoprefixer 这样的插件需要把你写的 css 样式适配不同的浏览器，那么这里要针对哪些浏览器呢，就是上面配置中所包含的。

而如果写在 autoprefixer 的配置中，那么会存在一个问题，万一其他第三方插件也需要浏览器的包含范围用于实现其特定的功能，那么就又得在其配置中设置一遍，这样就无法得以共用。所以在 package.json 中配置 browserslist 的属性使得所有工具都会自动找到目标浏览器。

当然，你也可以单独写在 .browserslistrc 的文件中：

至于它是如何去衡量浏览器的使用率和版本的，数据都是来源于 Can I Use。你也可以访问 browserl.ist/ 去搜索配置项所包含的浏览器列表，比如搜索 last 2 versions 会得到你想要的结果，或者在项目终端运行如下命令查看：

### 有哪些应用场景

autoprefixer、babel、postcss
