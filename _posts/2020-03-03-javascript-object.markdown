---
layout: post
title:  "javascript中的原型和原型链"
date:   2020-03-02 19:39:59 +0800
tag: [javascript, prototype, 原型]
---
[参考博客](https://github.com/mqyqingfeng/Blog/issues/2)
### 首先是对象
在javascript中，想要创建一个对象，最常见的做法是使用**构造函数**。
```javascript
function Person(){} // 最简单的构造函数
```
虽然函数里面啥也没有，但是使用`new`运算符就能创建一个对应的实例对象
```javascript
var person = new Person();
```
在chrome DevTools上看看这样通过这种方式创建出来的person实例长啥样

![person](/assets/person.png)

为什么一个空白的函数使用`new`之后能够创建出来这么一坨东西呢?请往下看

### 原型
每一个函数，都有一个`prototype`属性。这也是每一个函数都能作为构造函数的原因，`prototype`属性在创建函数的时候起到了关键性的作用。

而函数的`prototype`属性都指向一个**对象**,这个对象可以理解为模板，同一个构造函数生成的对象，内部都有这么一个链接链到这个模板上。请注意，是链接到这个模板上，而不是拥有这个模板的复制。


关于原型有一张很直观的图
![prototype](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype3.png)

1. Person函数的`prototype`属性指向一个对象，这个对象是Person函数所有生成的对象的**原型**(当然你如果在函数里显式返回了另一个obj那就是另外一回事了)
2. 原型这个对象,本身又有一个`constructor`属性指像构造函数。(当然也是在你不玩花的前提下)
3. 通过构造函数生成的**实例**，都具有的一个属性，叫__proto__，这个属性会指向该对象的原型

需要注意的是`__proto__`方法并不是EcmaScript语言规范，只是现代浏览器都实现了它。但是并不推荐使用，因为会[严重影响性能](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)。ES6开始可以使用`Object.getPrototypeOf()` 和 `Object.setPrototypeOf()`来操作和访问对象的原型。

一个有意思的等式: `Function.__proto__ === Function.prototype`

Function本身也是一种对象,只不过是内置的对象。因为是内置对象的原因，前述的这个等式成立本身只是为了阐述一种关系，但并不是Function调用自己生成了自己这种说法.

### 原型链
前面说过，原型本身也是一个对象。

既然是个对象，那么它肯定也是构造函数创造出来的

那么作为原型的这个对象, 它的`__proto__`属性又指向了哪里呢

##### 下面又到了盗图时间(实在是图画的太好)
![原型链](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype5.png)

构造函数的原型是通过最原始的方式`new Object()`创造出来的
所以
```javascript
Person.prototype.__proto__ === Object.prototype
```
而Object.prototype的__proto__就等于null了，null表示不应该有值。所以原型链追溯到这里就表示可以到此为止了。
