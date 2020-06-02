# css基础

工作三年多，发现css基础还是很薄弱，赶上求职之际，对重要的知识点进行总结学习

<!-- TOC -->

- [BFC](#bfc)
  - [什么是BFC](#什么是bfc)
  - [如何产生BFC](#如何产生bfc)
  - [BFC解决了什么问题](#bfc解决了什么问题)

<!-- /TOC -->

## BFC

### 什么是BFC

BFC(Block Formatting Contexts)直译为"块级格式化上下文"。

BFC就是页面上的一个隔离的渲染区域，可以理解为一个结界，容器里面的子元素不会在布局上影响到外面的元素，反之也是如此。

原则：如果一个元素具有BFC，内部子元素再怎么翻江倒海，都不会影响外部的元素

### 如何产生BFC

html元素
float的值不为none
overflow的值为auto、hidden、scroll
position的值不为relative和static
display的值为table-cell, table-caption, inline-block中的任何一个

### BFC解决了什么问题

1. 兄弟元素，margin重叠，外边距塌陷
2. 子元素浮动，导致父元素高度塌陷。给父元素设置BFC，清除浮动
3. 消除由于图片浮动导致的"文字环绕图片"现象，实现更健壮、更智能的自适应布局
