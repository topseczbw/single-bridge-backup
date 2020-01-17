# ts入门

2019/09/30 00:28

<!-- TOC -->

- [数据类型](#数据类型)
  - [es6](#es6)
  - [ts](#ts)
- [typescript 核心原则](#typescript-核心原则)
- [概念](#概念)
  - [什么是类型注解](#什么是类型注解)
  - [什么是类型断言](#什么是类型断言)
  - [可选属性 与 只读属性](#可选属性-与-只读属性)
  - [什么是接口](#什么是接口)
    - [接口如何实现继承](#接口如何实现继承)
    - [接口注意事项](#接口注意事项)
    - [如何绕过接口的类型检查](#如何绕过接口的类型检查)
  - [什么是索引签名](#什么是索引签名)
  - [如何区分const与readonly](#如何区分const与readonly)
  - [什么是混合类型](#什么是混合类型)
  - [接口继承类](#接口继承类)

<!-- /TOC -->

## 数据类型

### es6

1. string
2. number
3. undefined
4. boolean
5. null
6. function
7. array
8. symbol
9. object

### ts

1. **any**
2. never
3. void
4. 元组
5. **枚举**
6. **自定义类型**

## typescript 核心原则

对值所具有的数据结构进行类型检查

## 概念

### 什么是类型注解

语法：变量/函数:type

作用：类似于强类型语言的类型声明

```ts
let str: string = 'zbw'
```

### 什么是类型断言

有时候，我们比ts更了解某个值的详细信息，在你知道一个实体具有比它现有类型更具体的类型时。

使用类型断言，可以告诉编译器，“相信我，我知道自己在干什么”。

1. 使用尖括号

   ```ts
   let stringA: any = 'zbw'
   let stringALength: number = (<string>stringA).length
   ```

2. 使用as

   ```ts
   let stringA: any = 'zbw'
   let stringALength: number = (stringA as string).length
   ```

### 可选属性 与 只读属性

### 什么是接口

用于约束对象、类、函数的结构和类型

1. 对象

   ```ts
   interface Person {
     name?: string
     age?: number
   }
   ```

2. 函数

   ```ts
   interface searchFun {
     (val: string, sub: number): boolean
   }
   let searchFunction: searchFun
   searchFunction = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
   }
   ```

3. 类

   接口中描述一个方法，在类里实现它

   ```ts
   interface AAA {
     state: string
     open(name: string)
   }
   class BBB implements AAA {
     state: 'hello'
     open(name: string) {
       this.state = name
     }
   }
   ```

   类静态部分和实例部分的区别

#### 接口如何实现继承

一个接口可以继承多个接口，创建出多个接口的合成接口。

```ts
interface A {
  name: string
}
interface B extends A {
  age: number
}
let square = <B>{};
square.name = 'zbw'
square.age = 1

interface Square extends Shape, PenStroke {
    sideLength: number;
}

```

#### 接口注意事项

1. 传入的数据类型只要满足接口定义的必要条件即可通过检查
2. 接口定义为对象时，不会去检查属性的顺序

#### 如何绕过接口的类型检查

1. 使用类型断言（as 和 尖括号）两种方式
2. 使用字符串索引

   ```ts
   interface Person {
     name?: string
     [propName: string]: any
   }
   ```

3. 把对象字面量赋值给一个变量

   ```ts
    let squareOptions = { colour: "red", width: 100 };
    let mySquare = createSquare(squareOptions);
   ```

### 什么是索引签名

TypeScript支持两种索引签名：字符串和数字。

但是数字索引的返回值必须是字符串索引返回值类型的子类型。 这是因为当使用 number来索引时，JavaScript会将它转换成string然后再去索引对象。 也就是说用 100（一个number）去索引等同于使用"100"（一个string）去索引，因此两者需要保持一致。

### 如何区分const与readonly

最简单判断该用readonly还是const的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用 const，若做为属性则使用readonly。

### 什么是混合类型

使用一个接口A声明了一个函数类型，同时，这个接口又继承了一个声明了对象类型的接口B，那么此时接口A就是混合类型声明。

比如 `axios` 既可以是方法，又可以是对象，调用他的request、get等方法。

### 接口继承类
