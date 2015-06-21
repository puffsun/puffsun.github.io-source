---
layout: post
title: "JavaScript 属性描述符"
author: George Sun
date: 2015-06-16T07:33:45+08:00
comments: true
categories: ["javascript"]
keywords: codethoughts.info codethoughts javascript object descriptor 对象 属性描述符 编程语言 JS
description: JavaScript  Property Descriptor
---

## 什么是属性描述符
在[ES5](https://es5.github.io/)之前，JavaScript 没有内置的机制来指定或者检查对象某个属性(property)的特性(characteristics)，比如某个属性是只读(readonly)的或者不能被枚举(enumerable)的。但是在 ES5之后，JavaScript 被赋予了这个能力，所有的对象属性都可以通过属性描述符(Property Descriptor)来指定。

先来感受一下属性描述符：

{%highlight js lineno%}
var myObject = {
    a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// { value: 2, writable: true, enumerable: true, configurable: true }
{%endhighlight%}

这里可以看到，除了 `value` 之外，JavaScript 属性描述符有 `writable`, `enumerable`, 和 `configurable` 这三个特性。如果不指定的话，这三个特性都是 `true`。但我们也可以通过 `Object.defineProperty(...)` 函数来指定它们为我们想要的值。

{%highlight js lineno%}
var myObject = {};

Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
} );
// 上面的定义等同于 myObject.a = 2; 
// 所以如果不需要修改这三个特性，我们不会用 `Object.defineProperty`

myObject.a; // 2
{%endhighlight%}

下面让我们来看看属性描述符到底能为 JavaScript 带来哪些不一样的东西。

### 是否可写 (Writable)
首先，属性描述符可以用来控制对象的某个属性是否可写：

{%highlight js lineno%}
// "use strict";
var myObject = {};

Object.defineProperty( myObject, "a", {
    value: 2,
    writable: false, // 不可写!
    configurable: true,
    enumerable: true
} );

myObject.a = 3; // 写入的值将会被忽略

myObject.a; // 2
{%endhighlight%}

如果应用了 `strict mode` 的话，那么 `myObject.a` 将会抛出 `TypeError`，而不是仅仅忽略写入的值。ES5 还引入了对象属性的 [Getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) 和 [Setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set)，这里的 `writable: false`可以认为是和没有定义或者定义了没有任何操作的 `setters` 的情况大致等同。当然了，如果是 `strict mode` 下，需要在 `setters` 里面抛出 `TypeError` 来完全模拟 `writable: false` 的情形。

### 是否可配置 (Configurable)
这个特性用来描述对象的某个属性是否可以用 `Object.defineProperty(...)` 来重新配置：
{%highlight js lineno%}
var myObject = {
    a: 2
};

myObject.a = 3;
myObject.a;                 // 3

Object.defineProperty( myObject, "a", {
    value: 4,
    writable: true,
    configurable: false,    // 不可配置!
    enumerable: true
} );

myObject.a;                 // 4
myObject.a = 5;
myObject.a;                 // 5

Object.defineProperty( myObject, "a", {
    value: 6,
    writable: true,
    configurable: true,
    enumerable: true
} ); // TypeError

{%endhighlight%}
 
注意，一旦某个属性被指定为 `configurable: false`，那么就不能从新指定为 `configurable: true` 了，这个操作是单向，不可逆的。另外，这个特性还会影响 `delete` 操作的行为，来看一段代码：

{%highlight js lineno%}
var myObject = {
    a: 2
};

myObject.a;             // 2
delete myObject.a;
myObject.a;             // undefined

Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: false,
    enumerable: true
} );

myObject.a;             // 2
delete myObject.a;
myObject.a;             // 2
{%endhighlight%}

这里可以看到，一旦指定某个属性为 `configurable: false`，那么 `delete` 操作会被忽略。

### 是否可枚举 (Enumerable)
这个特性用来描述对象的某个属性是否在对象属性的枚举中出现，比如 `for..in` 循环中。来看这段代码：
{%highlight js lineno%}
var myObject = { };

Object.defineProperty(
    myObject,
    "a",
    // make `a` enumerable, as normal
    { enumerable: true, value: 2 }
);

Object.defineProperty(
    myObject,
    "b",
    // make `b` NON-enumerable
    { enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
    console.log( k, myObject[k] );
}
// "a" 2

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
{%endhighlight%}

这里可以看到，`enumerable: false` 使得该属性从对象属性枚举操作中被隐藏，但 `Object.hasOwnProperty(...)` 仍然可以检测到属性的存在。另外，`Object.propertyIsEnumerable(..)` 可以用来检测某个属性是否可枚举, `Object.keys(...)` 仅仅返回可枚举的属性，而 `Object.getOwnPropertyNames(...)` 则返回该对象上的所有属性，包括不可枚举的。


## 和属性描述符相关的操作

除了可以直接用 `Object.defineProperty(...)` 来指定属性描述符之外，JavaScript [ES5](https://es5.github.io/) 还提供了几个操作可以用来配置属性描述符。

### 对象常量 (Object Constant)
通过组合 `writable: false` 和 `configurable: false`，我们可以创建一个不能修改、重新定义或删除其属性的对象常量，比如：
{%highlight js lineno%}
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
    value: 42,
    writable: false,
    configurable: false
} );
{%endhighlight%}

这里对该对象属性的删除，修改操作会被忽略，你也不能再用 `Object.defineProperty(...)` 来重新配置该属性的特性。


### 禁止扩展 (Prevent Extensions)
如果希望阻止新的属性被加入到对象，可以通过调用 `Object.preventExtensions(...)` 来做到这一点：
{%highlight js lineno%}
var myObject = {
    a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
{%endhighlight%}

在 `strict mode` 下，行为稍有不同，对属性的赋值会抛出 `TypeError`, 而不是仅仅忽略赋值操作。

### 封装 (Seal)
可以通过 `Object.seal(...)` 来封装一个对象。在调用这个操作之后，对象上不能再添加新的属性，也不能重新定义属性描述符或者删除某个属性：
{%highlight js lineno%}
var myObject = {
    a: 2
};
Object.seal(myObject);
myObject.b = 'b';
console.log(myObject); // {a: 2}

myObject.a = 6;
console.log(myObject); // {a: 6}
{%endhighlight%}

事实上，`Object.seal(...)` 相当于调用了 `Object.preventExtensions(..)`，并设置现有的所有属性为 `configurable:false`。

### 冻结 (Freeze)
调用 `Object.freeze(...)` 可以创建一个被冻结的对象，这个对象拥有不能再被做任何修改或者删除属性的操作，效果相当于调用了 `Object.seal(...)` 并设置所有属性为 `writable: false`。
{%highlight js lineno%}
var myObject = {
    a: 2
};
Object.seal(myObject);
myObject.b = 'b';
console.log(myObject); // {a: 2}

myObject.a = 6;
console.log(myObject); // {a: 2}
{%endhighlight%}

需要注意的细节是，上述操作仅仅会设置对象的直接属性，而不会影响作为 `myObject` 对象的属性的对象的特性，比如：

{%highlight js lineno%}
var myObject = {
    innerObj: {
        a: 2
    },
    b: 3
}
console.log(myObject) // { innerObj: { a: 2 }, b: 3 }

Object.freeze(myObject);
myObject.b = 6;
console.log(myObject); // { innerObj: { a: 2 }, b: 3 }
myObject.innerObj.a = 6;
console.log(myObject) // { innerObj: { a: 6 }, b: 3 }

{%endhighlight%}
这里可以看到，即使调用了 `Object.freeze(...)`, 对 `innerObj` 属性的修改仍然成功了，对其他几个方法，比如 `Object.seal(...)` 或者 `Object.preventExtensions(...)` 也存在类似的情况，如果需要的话，可以对对象的属性递归调用上述方法。

## 资源

[You-Dont-Know-JS - this & object prototypes](https://github.com/getify/You-Dont-Know-JS/tree/master/this%20%26%20object%20prototypes)

[Working with objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects)

[Mozilla MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

[Annotated ECMAScript 5.1 Last updated: 2013-09-01](https://es5.github.io/)
