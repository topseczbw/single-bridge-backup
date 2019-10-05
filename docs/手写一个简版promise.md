# 手写一个简版promise

2019/10/05 02:20
<!-- TOC -->

- [promise 是什么](#promise-是什么)
- [promise 存在的价值](#promise-存在的价值)
- [promise 特点](#promise-特点)
- [如何手写一个简版 promise](#如何手写一个简版-promise)
  - [既然 promise 是一个构造函数/类，那么它 new 时可以传递哪些参数，这些参数又是做什么用的](#既然-promise-是一个构造函数类那么它-new-时可以传递哪些参数这些参数又是做什么用的)
  - [为了实现上述功能，每个 promise 实例应该具有哪些内部状态](#为了实现上述功能每个-promise-实例应该具有哪些内部状态)
  - [当 executor 函数只有同步时：2 > 1 ? resolve('回答正确') : throw new Error('原则性错误')](#当-executor-函数只有同步时2--1--resolve回答正确--throw-new-error原则性错误)
  - [当 executor 函数存在异步时：setTimeOut(() => resolve('async'), 2000)](#当-executor-函数存在异步时settimeout--resolveasync-2000)

<!-- /TOC -->
## promise 是什么

promise是一个构造函数/类

## promise 存在的价值

解决并发问题，多个异步任务的执行结果

解决链式问题，第二个接口依赖于第一个接口，解决回调地狱问题

## promise 特点

1. 每次new一个promise实例时，都需要传递一个执行器函数executor，并且执行器函数会立即执行

2. 执行器函数有两个参数 resolve reject
3. 每个promise都有一个状态标识，有三种状态类型：

   pending => resolve 成功了

   pending => reject 失败了

4. 状态可以改变，一旦改变不可以再次更改

   一旦成功了，不能再变成失败，一旦失败了，不能再变成成功
    （什么时候可以改变状态呢？ ）

5. 如果执行器函数在执行时，抛出异常如 `throw new Error('自定义错误')` ，那promise应该变成失败状态，会被.then时的 `onRejected` 处理错误的函数接收

## 如何手写一个简版 promise

### 既然 promise 是一个构造函数/类，那么它 new 时可以传递哪些参数，这些参数又是做什么用的

executor、resolve、reject

executor执行业务逻辑（同步/异步），在适当的时机调用，resolve/reject。

如数据成功返回时，调用resolve函数，将promise实例变成成功态，并将数据告诉promise实例，promise实例拿到数据后，先暂存起来，当promise实例被手动调用.then方法时，供then方法第一个处理成功信息的函数nFullfilled使用

如数据返回失败/代码报错时，调用reject函数，将promise实例变成失败态，并将失败原因告诉promise实例，promise实例拿到失败原因后，先暂存起来，当promise实例被手动调用.then方法时，供then方法第二个处理失败信息的函数onRejected使用

### 为了实现上述功能，每个 promise 实例应该具有哪些内部状态

1. 用户手动调用的resolve、reject函数不需要自己声明变量，所以调用的应该是new Promise() 时， Promise函数内部作用域的变量 => constructor 中的resolve和 reject
2. resolve函数可以改变promise状态，每个promise都有一个状态标识，标识当前状态 => this.status
3. 当resolve(data) / reject(reason)时，promise需要暂存外部告知的**数据**或**失败原因**，并且当外部手动调用then时，还需要用到 => this.value  this.reason
4. promise实例可以被手动调用then方法，处理成功时的数据或失败时的原因 => this.then()

### 当 executor 函数只有同步时：2 > 1 ? resolve('回答正确') : throw new Error('原则性错误')

由于executor函数会立即执行，所以如果该函数都是同步逻辑，promise状态会立即改变，数据会立即暂存，所以后面.then时，可以立即处理对应的数据。

### 当 executor 函数存在异步时：setTimeOut(() => resolve('async'), 2000)

但是当executor函数中有异步逻辑时，resolve函数不会马上执行，但是.then会在 new Promise 后马上执行。但此时promise还处于pending状态，所以我们需要在原型的then方法中，当状态是pending时，先将用户传来的处理数据/异常的方法暂存进一个队列，当调用resolve/reject，在从队列中遍历执行处理方法。此处使用发布订阅模式。

故promise增加onResolvedCallBacks、onRejectedCallBacks两个数组实例属性，暂存处理函数。

```js
console.log('——————————————————————————手写的promise——————————————————————')
const PENDING = 'pending'
const FULLFILLED = 'fullFilled'
const REJECTED = 'rejected'


class Promise {
  constructor(executor) {
    this.value = undefined;
    this.reason = undefined;
    this.status = PENDING
    // 两个队列 专门存成功的和失败的 为了处理异步 发布订阅模式
    this.onResolvedCallBacks = []
    this.onRejectedCallBacks = []

    let resolve = (value) => {
      // 只有pending状态时才可以改变promise的状态
      if (this.status === PENDING) {
        this.status = FULLFILLED
        this.value = value
        this.onResolvedCallBacks.forEach(fn => fn())
      }
    }
    let reject = (reason) => {
      if (this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedCallBacks.forEach(fn => fn())
      }
    }
    // 传递一个执行器 执行器会立即执行
    // 这里可能会发生异常
    try{
      executor(resolve, reject)
    } catch (e) {
      reject(e)
    }
  }

  // then方法会判断当前的状态
  then(onFullfilled, onRejected) {
    if (this.status === FULLFILLED) {
      onFullfilled(this.value)
    }
    if (this.status === REJECTED) {
      onRejected(this.reason)
    }
    // 异步 pending 时调用then   resolve 需要在 then 之后执行
    if (this.status === PENDING) {
      // ** todo
      this.onResolvedCallBacks.push(() => {
        onFullfilled(this.value)
      })
      this.onRejectedCallBacks.push(() => {
        onRejected(this.reason)
      })
    }
  }
}
module.exports = Promise

```
