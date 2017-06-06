---
layout: post
title: JavaScript中对象的属性
date: 2017-06-06 12:32:24.000000000 +09:00
categories: javascript
tags: javascript|对象属性
---
在JavaScript中，对象是==属性的无序集合==，通过对JS属性的理解可以更好地了解JS对象。

### 属性的构成与分类

在JS中，对像的属性是由名字(key)和一组特性(attribute)构成，其中:

**1. key是一个字符串（包括空字符串）**

**2. attribute是一些与之相关的值，主要是这4个：**

**值(value)**，可以是任意的JS值，也可以是一个getter或setter函数(或两者都有）。
当value为getter或setter时，那么我们称这个属性为“存取器属性(accessor property)”，其他时候则称作“数据属性(data property)”

**可写性(writable attribute）**，表时属性的值是否可以设置

**可枚举性(enumerable attribute)**，表时属性是否可能通过for/in循环返回

**可配置性(configurable attribute)**，表明属性是否可以删除或者修改


实际上，当属性是存取器属性时，是没有可写属性的，只有可枚举和可配置这两个属性。因此，可以这么说，对于一个数据属性，那么它有4个特性：值、可写性、可枚举性和可配置性，对于一个存取器属性，也有4个特性：get、set、可枚举和可配置性。


```
var person = {
    name: "Kaidi Yang",
    get familyName() {
        return this.name.split(" ")[1];
    },
    set familyName(val) {
        this.name = this.name.split(" ")[0] + " " + val;
    }
};

//输入name的属性描述
Object.getOwnPropertyDescriptor(person,'name')
out-{value: "Kaidi Yang", writable: true, enumerable: true, configurable: true}
Object.getOwnPropertyDescriptor(person,'familyName')
{enumerable: true, configurable: true, get: function, set: function}
```

ES5中也提供一个新的方法Object.defineProperty用来定义对象的属性：

```
var xxx = {};
Object.defineProperty(xxx, "k", {
    value: 1,
    writable: true,
    enumerable: true,
    configurable: true
});
console.log(xxx.k); // 1
var zzz = {};
Object.defineProperty(zzz, "t", {
    get: function() { return 2 },
    set: function(val) {
        console.log("get new val: " + val);
    },
    enumerable: true,
    configurable: true
});
console.log(zzz.t);  // 2
```


### 属性的访问

JS是一门基于原型继承的语言，当在访问对对象的属性时，会从自身开始一直检索整个原型链。


首先，一个JS对象具有“自有属性(own property)”，同时也有一些属性是从原型对象中继承来的。当我们访问一个对象xxx的属性k时，如果xxx中有k这个自有属性，那么，就会返回这个属性；如果对象xxx中没有属性k，那么就会继续在他的原型对象中查找属性k。如果原型对象中也没有，那么就会在这个原型对象的原型对象中查找，直到找到属性k或者查找到一个原型是null。

同样，当我们给对象xxx的属性k赋值时，如果xxx中已经有属性x（自有的）了，那么这个赋值操作就只改变这个自有属性的值；如果xxx中没有这个的自有属性k，那么赋值操作会给xxx==加上一个新的自有属性k==（如果xxx的的原型对象中有k，那么这个新加上的自有属性会覆盖原来的继承属性）。


上面是两是JS对象属性访问基本的规则，下面指出一些特殊的情况：

**不能给只读属性赋值，除非用defineProperty方法把可配置属性变成可写的**

**在覆盖原型对象中的同名属性时，如果此属性是只读的，那么覆盖会失败。比如下面的代码**


```
console.log(Object.getOwnPropertyNames(navigator))  // []  
navigator.userAgent = "hello";  
console.log(navigator.userAgent);  //Mozilla/5.0 (Windows NT 10.0; WO....
```

ps:
一个属性的configurable一旦被设置成false，那么:

- 它的枚举性和可配性就不能发生变化，
- 也不能从数据性变成存取性，或从存取性变成数据性，或者修改setter和getter
- 还不能将要可写性变从false变成true
- 只能把可写性从true变成false


