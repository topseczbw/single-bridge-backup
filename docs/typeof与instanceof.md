# typeof与instanceof

2020/02/01 19:01

<!-- TOC -->

- [typeof](#typeof)
- [instanceof](#instanceof)
  - [instanceof是如何运作的](#instanceof是如何运作的)
  - [实现一个instanceof](#实现一个instanceof)
  - [instanceof本质](#instanceof本质)
  - [如何实现让instanceof可以校验基本数据类型](#如何实现让instanceof可以校验基本数据类型)
- [判断数据类型其他方法](#判断数据类型其他方法)

<!-- /TOC -->
typeof 用于判断原始数据类型：number、string、null、undefined、boolean、symbol、以及function

instanceof 用于判断引用数据类型： Regexp、Object、Array

## typeof

- 当数据是方法时，返回'function'
- 当数据是正则时，返回'object'
- typeof null ==== 'object'

## instanceof

instanceof 只能校验某个对象是否是某个类的实例

### instanceof是如何运作的

例如: `A instanceof B`

取A.__proto__与B.prototype比较

如果相等，则返回true

如果不相等，则继续取A.__proto__.__proto__与B.prototype比较。直至左边为null

即沿着原型链的继承关系一直找，如果发现来同一个原型对象，则返回true，否则返回false

### 实现一个instanceof

```js
function instanceOf(A, B) {
  B = B.prototype;
  A = A.__proto__;

  while (true) {
    if (A === null) return false;

    if (A === B) return true;

    A = A.__proto__;
  }
}

console.log(instanceOf("zbw", String));
console.log(instanceOf("zbw", Object));
console.log(instanceOf("zbw", Array));
```

### instanceof本质

instanceof本质其实是调用类（方法）的静态方法`[Symbol.hasInstance]`对值进行校验

### 如何实现让instanceof可以校验基本数据类型

```js
class ValidateStr {
  static [Symbol.hasInstance](x) {
    return typeof x === "string";
  }
}

// 以下两种是等价的
console.log(ValidateStr[Symbol.hasInstance]("hello"));
console.log("hello" instanceof ValidateStr);
```

## 判断数据类型其他方法

1. Object.prototype.toString.call
2. instance.constructor
