# new、call、apply、bind原理

2020/02/01 18:36

<!-- TOC -->

- [new](#new)
- [call](#call)
- [apply](#apply)
- [bind](#bind)

<!-- /TOC -->
## new

1. new的过程是在干什么
2. 如何让生成的对象拥有实例属性和原型属性
3. 如果构造函数执行后返回对象，那么new的结果是什么

```js
function Person(name) {
  this.name = name;
}
Person.prototype.age = 10;

const instanceA = new Person("zbw");
console.log(instanceA.age);

function mockNew(constructor) {
  // 1. new的结果无非是创建一个新对象
  let obj = {};

  // 2. 让 传入的构造函数 以这个对象 为this，执行，为这个对象设置实例属性和原型属性
  // 方法一
  Object.setPrototypeOf(obj, constructor.prototype);
  // 方法二
  // obj.__proto__ = constructor.prototype

  const args = Array.from(arguments).slice(1);
  const result = constructor.call(obj, ...args);

  return result instanceof Object ? result : obj;
}
const instanceB = mockNew(Person, "zbw");
console.log(instanceB);
console.log(instanceB.age);
```

## call

1. call方法作用
2. 如何实现修改函数的this指向
3. 如何实现序列化传参

```js
  function say(a, b) {
    console.log("hello");
    console.log(this.name);
    console.log(a);
    console.log(b);
  }

  // 不能用箭头函数，箭头函数没有this，找不到实例方法
  Function.prototype.call = function(context) {
    // 1. 将传入的context转成一个对象
    console.log("手写call方法");
    context = context ? Object(context) : window;

    // 2. 如何改变方法的this指向
    // 利用 xxx.fn  时  fn的this指向xxx
    context.fn = this;

    // 3. 处理参数传给实例函数执行
    let args = Array.from(arguments).slice(1);

    context.fn(...args);
  };

  say.call(
    {
      name: "zbw"
    },
    1,
    2
  );
```

## apply

核心要点同call

```js
Function.prototype.apply = function(context) {
    console.log("手写apply方法");
    context = context ? Object(context) : window;

    context.fn = this;

    let args = Array.from(arguments)[1];

    context.fn(...args);
  };

say.apply(
  {
    name: "zbw"
  },
  [1, 2]
);
```

## bind

1. bind方法有哪些作用
2. 如何改变函数this指向
3. 如何实现参数部分持久化
4. ？如果bind返回的方法，被new形式调用。那么函数中的this指的是谁
5. ？new出来的结果可以找到原函数原型和实例上的属性

```js
function say(age, animal) {
  console.log("hello");
  console.log(`我的名字是${this.name}，今年${age}岁了，喜欢${animal}`);

  this.age = age;
  this.animal = animal;
}
say.prototype.sex = "男";
// const sayAdaptor = say.bind({ name: "zbw" });
// sayAdaptor();

// 浏览器环境正常， node环境中，报错

// 改变this指向，可以使用call、apply，但是低层都是使用一个三方对象作为中间层实现
Function.prototype.bind = function() {
  console.log("手写bind方法");

  const that = this;
  const context = [].shift.call(arguments);
  const bindArgs = Array.from(arguments);

  function bindFn() {
    const args = Array.prototype.slice.call(arguments);
    that.apply(this instanceof bindFn ? this : context, bindArgs.concat(args));
  }

  // Object.create()原理，利用中间层函数
  function Fn() {}
  Fn.prototype = this.prototype;
  bindFn.prototype = new Fn();

  return bindFn;
};

// 对于处理argument类数组对象时， Array.prototype.slice.call === [].slice.call

const bindFn = say.bind({ name: "zbw" }, "17");
bindFn("猫");
```
