# node常用包

<!-- TOC -->

- [node常用包](#node%e5%b8%b8%e7%94%a8%e5%8c%85)
  - [I/O相关](#io%e7%9b%b8%e5%85%b3)
    - [stat](#stat)
    - [Crypto](#crypto)
  - [http相关](#http%e7%9b%b8%e5%85%b3)
    - [mime](#mime)

<!-- /TOC -->

## I/O相关

### stat

```js
const statObj = stat(pathName)

// 文件最新修改时间
statObj.ctime
```

### Crypto

文件加密

## http相关

### mime

`Mime.getType()`，根据文件名称，获取 `content-type`, 设置响应头
