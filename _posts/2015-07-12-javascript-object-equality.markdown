---
layout: post
title: 判定 JavaScript 对象是否相等 
date: 2015-07-12T21:20:31+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: Object Equality 对象 相等 codethoughts.info codethoughts javascript 编程语言 JS == === 相等性 
description: JavaScript Object Equality

---

JavaScript 中有多个比较相等性的运算符：`==`、 `===` 和 ES2015 引入的 `Object.is(...)`，它们的行为并不相同，这给初学者带来了很多困惑。不过我们今天的主题并不是讨论这三者的差别，而是另外一个很常用的主题，JavaScript 对象相等性的判别。

## JavaScript 如何判定对象相等性

首先来看一个示例：

{%highlight js lineno%}
var obj1 = {
    foo: "FOO",
    bar: "BAR"
};

var obj2 = {
    foo: "FOO",
    bar: "BAR"
};

// Outputs: false
console.log(obj1 === obj2);
{%endhighlight%}

这里我们使用 `===` 来判定对象的相等性，`===` 在判定相等性的时候会考虑对象的类型，而且不会有隐式的类型转换。从输出可以看到，即便两个对象内容完全相同，二者也并不相等。事实上，即便我们用 `==` 来判定，结果也是一样的：

{%highlight js lineno%}
// Outputs: false
console.log(obj1 == obj2);
{%endhighlight%}

其中的原因是 JavaScript 在判断相等性的时候对原生类型和对象区别对待。如果是原生类型，比如说字符串和数字，那么 JavaScript 根据值来比较；如果是对象，那么要根据对象的引用来判断，也就是对象在内存中引用的地址，我们来验证一下：

{%highlight js lineno%}
var obj1 = {
    foo: "FOO",
    bar: "BAR"
};

var obj2 = {
    foo: "FOO",
    bar: "BAR"
};

var obj3 = obj1;

// Outputs: false
console.log(obj1 === obj2);

// Outputs: true
console.log(obj1 === obj3);
{%endhighlight%}

这里我们可以看到，对象 `obj1` 和 `obj3` 引用了同一个内存地址，他们是相等的，而 `obj2` 和 `obj1` 引用了不同的内存地址，他们并不相等。

## 区分对象值相等和引用相等

基于上面的结论，在 JavaScript 代码中判定对象是否相等之前，必须明确我们需要判断是需要严格的内存引用地址相等还是两个对象的内容相等。两个对象内存引用相等表示两个对象指向的是同一个内存地址，本质上是同一个对象，只是两个引用地址而已，而内容相等则宽松得多，也是我们实际的代码中最常用到的。

但是，很不幸，JavaScript 没有原生的为我们提供判断两个对象内容是否相等的工具，我们需要自己实现它或者使用第三方类库。这和我们写 Java 代码的时候，实现对象的 `equals(...)` 方法在概念上是一致的，但是做法则完全不同。鉴于 JavaScript 要比 Java 灵活得多，我们也有更多的办法来达到我们的目的。

## 判断两个对象内容是否相等

这里我们来看一个实现判断两个对象内容是否相等的方法：

{%highlight js lineno%}
function isEquivalent(a, b) {
    // 获取对象属性的所有的键
    var aProps = Object.getOwnPropertyNames(a);
    var bProps = Object.getOwnPropertyNames(b);

    // 如果键的数量不同，那么两个对象内容也不同
    if (aProps.length != bProps.length) {
        return false;
    }

    for (var i = 0, len = aProps.length; i < len; i++) {
        var propName = aProps[i];

        // 如果对应的值不同，那么对象内容也不同
        if (a[propName] !== b[propName]) {
            return false;
        }
    }

    return true;
}

// Outputs: true
console.log(isEquivalent(obj1, obj2));
{%endhighlight%}

请注意，我们这里用到了 `Object.getOwnPropertyNames(a)`，这是由 ES5 引入的，如果你还需要考虑过时的浏览器，比如 IE8 :(，那么你可以考虑使用 [es5-shim](https://github.com/es-shims/es5-shim)。如果你想了解更多关于 ES5 浏览器兼容性的问题，可以参考 [ES5 浏览器兼容对照表](http://kangax.github.io/compat-table/es5/)。

从上面的代码我们可以看到，判断两个对象内容是否相同，我们需要遍历对象的所有属性，并依次判断每个键对应的值是否相同。这里的实现并不严谨，有很多情况还无法处理。

* 对象嵌套的情况，比如一个对象属性的值是另外一个对象；
* 对象属性的值是 `NaN`，我们知道 JavaScript 的 `NaN` 和自己是不相同的，比如 `console.log(NaN === NaN) // false`
* ...

这里我们来看一个产品级的实现，代码来自 [Lo-Dash 库](https://github.com/lodash/lodash/blob/master/lodash.src.js)：

{%highlight js lineno%}
    function baseIsEqualDeep(object, other, equalFunc, customizer, isLoose, stackA, stackB) {
      var objIsArr = isArray(object),
          othIsArr = isArray(other),
          objTag = arrayTag,
          othTag = arrayTag;

      if (!objIsArr) {
        objTag = objToString.call(object);
        if (objTag == argsTag) {
          objTag = objectTag;
        } else if (objTag != objectTag) {
          objIsArr = isTypedArray(object);
        }
      }
      if (!othIsArr) {
        othTag = objToString.call(other);
        if (othTag == argsTag) {
          othTag = objectTag;
        } else if (othTag != objectTag) {
          othIsArr = isTypedArray(other);
        }
      }
      var objIsObj = objTag == objectTag && !isHostObject(object),
          othIsObj = othTag == objectTag && !isHostObject(other),
          isSameTag = objTag == othTag;

      if (isSameTag && !(objIsArr || objIsObj)) {
        return equalByTag(object, other, objTag);
      }
      if (!isLoose) {
        var objIsWrapped = objIsObj && hasOwnProperty.call(object, '__wrapped__'),
            othIsWrapped = othIsObj && hasOwnProperty.call(other, '__wrapped__');

        if (objIsWrapped || othIsWrapped) {
          return equalFunc(objIsWrapped ? object.value() : object, othIsWrapped ? other.value() : other, customizer, isLoose, stackA, stackB);
        }
      }
      if (!isSameTag) {
        return false;
      }
      // Assume cyclic values are equal.
      // For more information on detecting circular references see https://es5.github.io/#JO.
      stackA || (stackA = []);
      stackB || (stackB = []);

      var length = stackA.length;
      while (length--) {
        if (stackA[length] == object) {
          return stackB[length] == other;
        }
      }
      // Add `object` and `other` to the stack of traversed objects.
      stackA.push(object);
      stackB.push(other);

      var result = (objIsArr ? equalArrays : equalObjects)(object, other, equalFunc, customizer, isLoose, stackA, stackB);

      stackA.pop();
      stackB.pop();

      return result;
    }
    
    function baseIsEqual(value, other, customizer, isLoose, stackA, stackB) {
      if (value === other) {
        return true;
      }
      if (value == null || other == null || (!isObject(value) && !isObjectLike(other))) {
        return value !== value && other !== other;
      }
      return baseIsEqualDeep(value, other, baseIsEqual, customizer, isLoose, stackA, stackB);
    }
    
    function isEqual(value, other, customizer) {
      customizer = typeof customizer == 'function' ? customizer : undefined;
      var result = customizer ? customizer(value, other) : undefined;
      return result === undefined ? baseIsEqual(value, other, customizer) : !!result;
    }
{%endhighlight%}

代码很长，我这里就不一一解释了，列举出来只是为了说明实现一个健壮的对象相等性判定函数并不容易，我们最好还是使用 [Lo-Dash](http://lodash.com/docs#isEqual) 或者 [Underscore](http://underscorejs.org/#isEqual) 这样经过亿万产品检验的类库来做这件事，不用重新发明轮子。他们实现的 API 是一样的 `isEqual(...)`，我们知道 Lo-Dash 和 Underscore 的 API 是兼容的。关于 Lo-Dash 的诞生还有一个和 Underscore 有关的有趣故事，我们得到的教训就是 `别惹大神，否则他会干翻你`！

把 Lo-Dash 或者 Underscore 应用到我们前面的代码中：

{%highlight js lineno%}
// Outputs: true
console.log(_.isEqual(obj1, obj2));
{%endhighlight%}

和我们的预期完全一致。

## 资源

[Object Equality in JavaScript](http://adripofjavascript.com/blog/drips/object-equality-in-javascript.html)

[Lo-Dash](https://lodash.com/)

[Underscore](http://underscorejs.org/)

[How to determine equality for two JavaScript objects?](http://stackoverflow.com/questions/201183/how-to-determine-equality-for-two-javascript-objects)



