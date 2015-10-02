---
layout: post
title: AngularJS providers
date: 2015-10-01T17:43:11+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: AngularJS angular.js provider Unit Test Angular value constant service factory
description: AngularJS Provider

---

之前的一篇文章 [AngularJS 单元测试](http://codethoughts.info/javascript/2015/09/06/angularjs-apps-unit-testing/) ，我解释了如何为AngularJS应用写单元测试，其中包括如何为Controller，Service，Directive，以及 AngularJS 应用中的 HTTP 请求编写单元测试。这篇文章我们来看看 AngularJS 的 Provider，以及如何为它编写单元测试。

## 简介
AngularJS 新手常犯的一个错误是 Controller 中被塞入了过多的业务逻辑，有些情况下，你甚至可以看到大部分业务逻辑和数据序列化的代码被放在 Controller 中。这是不合适的，AngularJS 的 Controller 应该是薄薄的一层，你的大部分业务逻辑和数据序列化代码应该放在 Service 中。我们知道，AngularJS 的 Controller 每次调用（刷新页面或者改变路由）会生成一个全新的实例，并在调用完毕以后被垃圾回收。而 Service 则是一个单例（每个 $injector 一个唯一的实例）。从提高应用程序的性能的角度来说，我们也应该把绝大部分的逻辑放在 Service 中，从而避免不必要的垃圾回收。

AngularJS 提供了5个创建 Service 的函数：

* provider
* factory
* service
* constant
* value

这里有的人可能会对 Service 这个术语产生混淆，这个我觉得是 AngularJS 的问题（AngularJS 也不是十全十美的）。以上几个函数创建的用户自定义对象被统称为Service，而其中的 service 方法又独立的创建一种 Service。Service 是 AngularJS应用程序中一层的统称，而 service 方法是创建这一层的一种方式。

那为什么本文标题又叫做 AngularJS provider 呢？因为 value、constant、factory 和 service 方法都仅仅是 provider 的语法糖。我们来看一段 [AngularJS 源码片段](https://github.com/angular/angular.js/blob/master/src/auto/injector.js#L655-680)：

{%highlight js linenos%}
////////////////////////////////////
  // $provider
  ////////////////////////////////////

  function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      provider_ = providerInjector.instantiate(provider_);
    }
    if (!provider_.$get) {
      throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
    }
    return providerCache[name + providerSuffix] = provider_;
  }

  function factory(name, factoryFn, enforce) {
    return provider(name, {
      $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
    });
  }

  function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
  }

  function value(name, val) { return factory(name, valueFn(val), false); }

  function constant(name, value) {
    assertNotHasOwnProperty(name, 'constant');
    providerCache[name] = value;
    instanceCache[name] = value;
  }

  function decorator(serviceName, decorFn) {
    var origProvider = providerInjector.get(serviceName + providerSuffix),
        orig$get = origProvider.$get;

    origProvider.$get = function() {
      var origInstance = instanceInjector.invoke(orig$get, origProvider);
      return instanceInjector.invoke(decorFn, null, {$delegate: origInstance});
    };
  }
{%endhighlight%}

从上面的代码片段你可以看到，所有其他的几个 function 都仅仅是 provider 的语法糖而已。既然如此，那么上面几个函数的又分别是什么？它们又有什么区别？

## value

value 的作用非常简单，它用来创建一个变量名，这个变量被绑定到一个值，并且可以把这个变量注入到任何地方去，value 的用法如下：

{%highlight js%}
module.value('name', any_value);
{%endhighlight%}

上面代码片段中，any_value 的值可以是 string, number, array, object 或者是 function。

## constant

constant 和对象区别在于：用 constant 创建的常量可以用在 AngularJS 的应用配置阶段（configuration phase），也就是 AngularJS 应用的启动阶段，而 value 创建的值仅仅可以在 AngularJS 应用的运行时使用。其语法如下：
{%highlight js%}
module.constant('const_name', const_value);
{%endhighlight%}

上面代码片段中，const_value 的值可以是 string, number, array, object 或者是 function。

## factory

factory 是用来创建 AngularJS Service 最常见的方式，首先你创建一个对象，在对象上创建属性和函数，并返回该对象。当你通过依赖注入把该对象注入到 Controller 中或者其他 Service 中，你就可以使用该对象上的属性和函数。我们来看一个例子：

{%highlight js linenos%}
var app = angular.module('App', []);

app.factory('ookFactory', ['$log', function($log) {
    "use strict";
    return {
        ook: function() {
            $log.warn('Ook.');
        }
    };
}]);

app.controller('myFactoryCtrl', ['ookFactory', function(ookFactory) {
	'use strict';
	ookFactory.ook();
}]);
{%endhighlight%}

在上面的代码片段中，我们把 `ookFactory` service 注入到 `myFactoryCtrl` 中，从而可以存取 `ookFactory` 中的函数。关于如何为 factory 写单元测试请参考 [AngularJS 单元测试](http://codethoughts.info/javascript/2015/09/06/angularjs-apps-unit-testing/)，我们这里不再赘述。

## service

service 和 factory 最大的区别在于 service 返回的函数将会被执行 `new` 操作来创建一个对象实例，当然了，这个实例对于每个 $injector 实例是全局唯一的。来看一个实例：

{%highlight js linenos%}
var module = angular.module('myapp', []);
 
module.service('userService', function(){
    this.users = ['George', 'Georgy', 'John'];
});
{%endhighlight%}

我们可以看到这里使用了 this，通过前一篇文章：[寻找 JavaScript 的 this](http://codethoughts.info/javascript/2015/06/04/javascript-find-the-lost-this/) 我们知道在通过 `new` 创建一个对象之后，this 将会指向新创建的对象，因此该对象拥有了 users 数组。为 service 写单元测试和为 factory 写单元测试没有区别，可以参考我写的同一篇文章。

## provider

这才是我们这篇文章的重头戏。通过前文我们知道，value, constant, service 和 factory 都仅仅是 provider 的语法糖，另外，constant 可以在 AngularJS 应用的配置阶段（app.config(function() {...})）使用，provider 可以在应用配置阶段来配置其行为，其他三个都不行。来看一个 provider 的代码实例：

{%highlight js linenos%}
var app = angular.module('myProviders', []);

app.provider('coffeeMaker', function() {
  var useFrenchPress = false;
  this.useFrenchPress = function(value) {
    if (value !== undefined) {
      useFrenchPress  = !!value;
    }

    return useFrenchPress;
  };

  this.$get = function () {
    return {
      brew: function() {
        return useFrenchPress ? 'Le café.': 'A coffee.';
      }
    };
  };
});
{%endhighlight%}

我们可以在应用程序启动阶段来配置 provider 实例的行为：

{%highlight js linenos%}
var app = angular.module('myProviders');

app.config(function(myProvider) {
	myProvider.useFrenchPress(true);
});
{%endhighlight%}

从以上代码可以看到，provider 必须定义一个叫做 `$get` 的函数，它将在 provider 被实例化的时候被调用，并返回该 provider 的对象实例。在此之前（应用程序启动阶段），我们可以通过 `.config()` 函数来配置它的行为。

## 为 provider 写单元测试

单元测试代码如下：

{%highlight js linenos%}
describe('coffee maker provider', function() {
  var coffeeProvider = undefined;

  beforeEach(function() {
    // Here we create a fake module just to intercept and store the provider
    // when it's injected, i.e. during the config phase.
    angular.module('dummyModule', function() {})
      .config(['coffeeMakerProvider', function(coffeeMakerProvider) {
        coffeeProvider = coffeeMakerProvider;
      }]);

    module('myProviders', 'dummyModule');

    // This actually triggers the injection into dummyModule
    inject(function(){});
  });

  describe('with french press', function() {
    beforeEach(function() {
      coffeeProvider.useFrenchPress(true);
    });

    it('should remember the value', function() {
      expect(coffeeProvider.useFrenchPress()).to.equal(true);
    });

    it('should make some coffee', inject(function(coffeeMaker) {
      expect(coffeeMaker.brew()).to.equal('Le café.');
    }));
  });

  describe('without french press', function() {
    beforeEach(function() {
      coffeeProvider.useFrenchPress(false);
    });

    it('should remember the value', function() {
      expect(coffeeProvider.useFrenchPress()).to.equal(false);
    });

    it('should make some coffee', inject(function(coffeeMaker) {
      expect(coffeeMaker.brew()).to.equal('A coffee.');
    }));
  });
});
{%endhighlight%}

可以看到，除了可以在应用程序启动之前配置 provider 的行为之外，测试 provider 和测试其他的 service 并没有太大的区别。


## 资源
[AngularJS: Factory vs Service vs Provider](http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/)

[AngularJS GitHub](https://github.com/angular/angular.js)

[AngularJS: The Provider Subsystem](http://anandmanisankar.com/posts/angularjs-provider-subsystem/)

[AngularJS 单元测试](http://codethoughts.info/javascript/2015/09/06/angularjs-apps-unit-testing/)

[An Introduction To Unit Testing In AngularJS Applications](http://www.smashingmagazine.com/2014/10/introduction-to-unit-testing-in-angularjs/)




