# webpack常见问题
<!-- TOC -->

- [源码暴露](#源码暴露)

<!-- /TOC -->
## 源码暴露

项目发布上线后，在控制台source选项中的webpack://文件夹中可以看到项目源码问题

![源码暴露](../assets/源码暴露.jpg)

解决方案

关闭webpack中的sourceMap功能，即 `devtool` 选项

vue-cli3中，则许关闭vue.config.js中的 `productionSourceMap` 配置
