---
layout: post
title: Object.assign()
date: 2018-01-02 14:32:24.000000000 +09:00
categories: javascript
tags: javascript-方法-笔记
---
### Object.assign()

这是ES6新添加的方法，Object.assign()用来复制源对象的所有可枚举属性复制到目标对象中，方法返回目标对象。语法如下：


```
Object.assign(target, ...source);
```

如果源对象和目标对象的属性的key相同，目标对象的属性将会被源对象中的属性覆盖。对于多个源对象来说，如果有相同的key，右边的属性将覆盖左边的属性。这个方法只能将==源对象的可枚举属性和自己的属性复制==给目标对象。




es5中常用的属性复制的方法
```

//如果Object.assign方法存在，则直接使用es6中的属性复制。否则使用后面的函数，实现属性的复制。
var _extends = Object.assign || function(target) {
    for (var i = 1; i < arguments.length; i++) {
        var source = arguments[i];
        for (var key in source) {
            if (Object.prototype.hasOwnProperty.call(source, key)) { target[key] = source[key]; }
        }
    }
    return target;
};


target=_extends({},props) 等同于 Object.assign(target,props)
```

### Object.create()
通过指定的原型对象和属性，创建一个新的对象。语法如下：


```
Object.create(proto, [,. propertiesObject]);
```
proto：一个对象，应该是新创建的对象的原型。

propertiesObject： 该参数对象是一组属性与值

抛出异常：如果 proto 参数不是 null 或一个对象值，则抛出一个 TypeError 异常。

Object.create()最直接的作用是基于一个对象创建新的对象，更多时候用在了原型链继承上，先来看看 JavaScript中创建对象的几种方

**1. 对象字面量**
```
var o = {a: 1};

// o这个对象继承了Object.prototype上面的所有属性
// 所以可以这样使用 o.hasOwnProperty('a').
// hasOwnProperty 是Object.prototype的自身属性。
// Object.prototype的原型为null。
// 原型链如下:
// o ---> Object.prototype ---> null

var a = ["yo", "whadup", "?"];

// 数组都继承于Array.prototype
// (indexOf, forEach等方法都是从它继承而来).
// 原型链如下:
// a ---> Array.prototype ---> Object.prototype ---> null

function f(){
  return 2;
}

// 函数都继承于Function.prototype
// (call, bind等方法都是从它继承而来):
// f ---> Function.prototype ---> Object.prototype ---> null
```
**2. 构造函数**

在 JavaScript 中，构造器其实就是一个普通的函数。当使用 new 操作符 来作用这个函数时，它就可以被称为构造方法（构造函数）。如果没有 new 关键字而是直接调用的话，相当于是在当前作用域上调用，此时函数中如果有 this 的话，this 指向的是当前作用域。


```
function Graph() {
  this.vertexes = [];
  this.edges = [];
}

Graph.prototype = {
  addVertex: function(v){
    this.vertexes.push(v);
  }
};

var g = new Graph();
// g是生成的对象,他的自身属性有'vertices'和'edges'.
// 在g被实例化时,g.[[Prototype]]指向了Graph.prototype.
Graph();
console.log(window.vertexes); // 在全局作用域中调用，意外地增加了全局变量
```
**3. 使用 Object.create()**


```
var a = {a: 1};
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (继承而来)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined, 因为d没有继承Object.prototype
```


**4. ES6 中的 Class 关键字**

ES6 引入了一套新的关键字用来实现 class。这是一个语法糖，其本质还是基于原型的。这些新的关键字包括 class, constructor, static, extends, 和 super。关于 Class的使用，回头再开一篇文章深入学习。

