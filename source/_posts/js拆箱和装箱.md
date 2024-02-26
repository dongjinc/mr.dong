---
title: 拆箱和装箱
layout: post
excerpt: 在JavaScript中, 存在Number、String、Boolean基本包装类型. 与基本类型number、string、boolean有什么关系呢?
---

## 简介
### 装箱
- 所谓装箱, 是指将基本类型数据变成对应引用类型的操作, 又分为显式装箱和隐式装箱

- 隐式装箱: 对基本类型数据进行数据操作(.indexOf/.substring)时, 引擎在执行时,会自动将其转换成基本包装类型对象,读取其继承自对象上的属性(方法), 实现数据操作
```js
  const demo1 = 'I Love JavaScript';
  demo1.substring(0, 1) // I
```
- 首先 ⚠️: 基本类型是不可以添加属性(方法);
```js
  const demo2 = 'I Love JavaScript';
  demo2.name = 'Yong';
  demo2.sayHi = function(){
    console.log('Hi Yong')
  }
  demo2.name // undefined;
  demo2.sayHi() // Uncaught TypeError: demo2.sayHi is not a function
```
- 那上面代码中, .substring 是从何而来的 ? 我们接下来看看隐式装箱的魅力
```js
  const demo3 = 'I Love JavaScript'; // 隐式装箱
  demo3.substring(0, 1) // I
  // 该代码执行步骤是 : 
  //1.先创建一个String类型的一个实例;
  //2.调用实例中原型链上的substring方法
  //3.销毁这个实例
  // 代码演示:
  const demo3 = String('I Love JavaScript');
  demo3.substring(0, 1);
  demo3 = null // 通过null来表示实例销毁
```
- 所以，我们在基本类型值上可以使用方法（比如string的substring等），是因为有「隐式装箱」操作
- 隐式装箱在读取一个基本类型数据时, 会在后台创建一个与之对应的基本包装类型对象. 在这个基本类型上调用的方法, 都是来源自这个后台创建的基本包装类型对象上. 这个基本包装类型的对象是临时的，它只存在于方法调用那一行代码执行的瞬间，执行方法后立即被销毁. 这也是在基本类型上添加属性和方法会不识别或报错的原因了。

- 显式装箱: 通过使用基本包装类型对象对基本类型数据进行包装, 简言之 通过Number、String、Boolean构造调用方式实现对基本数据类型进行包装
```js
    const demo1 = new String('I Love JavaScript');
    demo1.sayHi = function(){
        console.log('Hi Yong')
    }
    demo1.sayJob = function(){
        console.log('Hi Job')
    }
```
- 由于通过new方式调用了String构造函数生成实例, 对其设置的属性都绑定在这个实例上.

### 拆箱
- 所谓拆箱, 是将引用类型对象转换成基本数据类型. 通常通过引用类型的valueOf和toString来实现.

- 什么是ToPrimitive ?
- JavaScript 对象转换到基本类型值时，会使用 ToPrimitive 算法，这是一个内部算法，是编程语言在内部执行时遵循的一套规则。
- 通过定义@@toPrimitive方法来覆盖默认行为
```js
    const demo5 = [1, 2, 3];
    demo5[Symbol.toPrimitive] = function() {
        return 'custom';
    }
```

- 实现原理: 拆箱过程中, 内部抽象 ToPrimitive . 该操作接受两个参数，第一个参数是要转变的对象，第二个参数 PreferredType 是对象被期待转成的类型。第二个参数不是必须的，默认该参数为 number，即对象被期待转为数字类型。有些操作如 String(obj) 会传入 PreferredType 参数。有些操作如 obj + " " 不会传入 PreferredType。 默认情况下，ToPrimitive 先检查对象是否有 valueOf 方法，如果有则再检查 valueOf 方法是否有基本类型的返回值。如果没有 valueOf 方法或 valueOf 方法没有返回值，则调用 toString 方法。如果 toString 方法也没有返回值，产生 TypeError 错误。
PreferredType 影响 valueOf 与 toString 的调用顺序。如果 PreferrenType 的值为 string。则先调用 toString ,再调用 valueOf。

<!-- https://www.jianshu.com/p/d66cf6f711a1 -->
<!-- https://segmentfault.com/a/1190000016325587 -->
<!-- https://262.ecma-international.org/6.0/#sec-well-known-symbols -->
<!-- https://sinaad.github.io/xfe/2016/04/15/ToPrimitive/ -->
<!-- https://juejin.cn/post/7040094995033882661 -->

