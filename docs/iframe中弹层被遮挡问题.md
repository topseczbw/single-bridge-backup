# iframe中弹层被遮挡问题

2019/09/08 13:04

当我们使用iframe时，如果iframe内部的弹层尺寸过大，超出iframe的部分会被隐藏

## 解决方案

1. 可以通过postMessage向外部传递数据，在调用方打开弹层，用于简单的弹层交互场景
2. 在调用方iframe元素外部包裹一个wrapper，当iframe内打开弹层时，使用postMessage通知外部修改这个wrapper样式为全屏显示，此时弹层正常显示，但iframe内容也会全屏，所以需要外部把iframe的尺寸和相对于屏幕的位置告知服务方，服务方修改iframe内容的样式
