---
layout: post
title: AngularJS 单元测试
date: 2015-09-06T10:41:20+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: 框架 codethoughts.info codethoughts javascript 编程语言 JS Angular AngularJS angular.js Unit test Testing UT Karma
description: AngularJS Apps Unit Testing
---

## 简介

AngularJS 起初由 Google 的一位测试工程师 Misko Hevery编写。在设计之初，易于为应用程序编写测试就是 AngularJS 的核心设计目标之一。 我们知道，在为应用程序编写测试的时候，最困难的部分就是管理应用程序外部依赖，AngularJS 通过模块化和依赖注入使得管理依赖变成一件轻松的事情。另外，AngularJS 可以轻松和第三方的测试框架和类库集成，比如：Mocha/Chai/Sinon，Jasmine，这也是为前端应用开发测试代码的另一个利好，你没必要再去熟悉新的框架或者类库。我在另一篇博客中介绍了如何使用 [用 Mocha/Chai/Sinon 测试 JavaScript 代码](http://codethoughts.info/javascript/2015/07/18/javascript-bdd-with-mocha-chai-sinon/)，它着重介绍了如何测试 JavaScript 代码。然而编写 AngularJS 应用程序的单元测试又需要学习一些新的知识，这也是这篇文章要讨论的主题。


本文假定你知道什么是单元测试，熟悉 AngularJS 的基本概念，比如 Module，Controller，Service，Directive，依赖注入等。如果这时候你仍然不清楚，可以参考[关于 Angular.js 你需要知道的知识](http://codethoughts.info/angular.js/2015/05/15/things-i-wish-i-were-told-about-angular-js/) 和 [用 Mocha/Chai/Sinon 测试 JavaScript 代码](http://codethoughts.info/javascript/2015/07/18/javascript-bdd-with-mocha-chai-sinon/) 这两篇文章，或者自行 [Google](www.google.com)。

AngularJS 的测试又可以大致分为单元测试和 E2E 测试，本文将不会涉及 E2E 测试，感兴趣的同学可以参考 [Protractor 主页](https://angular.github.io/protractor/#/)。

本文中测试代码将使用 Mocha/Chai/Sinon 测试类库来编写。当然了，你也可以使用 Jasmine，两者的功能基本一致，只是语法略有区别。Jasmine 是一栈式的测试框架，而 Mocha/Chai/Sinon 的组合方案更具灵活性。

## 设置测试环境

设置一个基本的测试环境需要安装几个 NPM 包，我们一一过一遍。

### 安装/配置 Karma

在这篇文章里我使用 [Karma](http://karma-runner.github.io/0.13/index.html) 来驱动 AngularJS 的单元测试。从它主页的简介可以看出，Karma 可以用来启动一个应用服务器，并在其中运行测试代码和需要被测试的源码，并把测试结果打印到命令行。另外它还具有一些对 TDD 很有帮助的功能，比如监控文件夹下的文件变动，并自动运行对应的测试代码。安装 Karma 很简单：

{%highlight bash%}
$ mkdir angularjs-unit-testing && cd $_
$ npm init ## use default configuration
$ npm install -g karma-cli
$ npm install -D karma karma-mocha karma-chai karma-phantomjs-launcher karma-sinon-chai karma-coverage
$ karma init
{%endhighlight%}

首先我们安装了一个全局的 Karma 命令行工具，然后安装了所需的依赖，最后，运行 `karma init` 来生成Karma 配置文件。Karma 启动的时候，会默认寻找 `karma.conf.js` 配置文件，我们要做的就是在项目根目录下运行 `karma init`，并填入我们所需要的配置。关于每个配置项的含义，可以参考 [Karma 文档](http://karma-runner.github.io/0.8/config/configuration-file.html)。

我们的 Karma 配置文件内容如下：

{%highlight js lineno%}
// Karma configuration
// Generated on Sun Sep 06 2015 11:33:08 GMT+0800 (CST)
module.exports = function(config) {
  config.set({

    // base path used to resolve all patterns (e.g. files, exclude)
    basePath: '',

    // frameworks to use
    frameworks: ['mocha', 'sinon-chai'],

    // list of files / patterns to load in the browser
    files: [
      'bower_components/angular/angular.js',
      'bower_components/angular-mocks/angular-mocks.js',
      'src/*.js',
      'test/*.mocha.js'
    ],

    // list of files to exclude
    exclude: [],

    // preprocess matching files before serving them to the browser
    preprocessors: {
      'src/*.js': ['coverage']
    },

    coverageReporter: {
      type: 'text-summary',
      dir: 'coverage/'
    },

    // test results reporter to use
    reporters: ['progress', 'coverage'],

    // web server port
    port: 9876,

    // enable / disable colors in the output (reporters and logs)
    colors: true,

    // level of logging
    logLevel: config.LOG_INFO,

    // enable / disable watching file and executing tests on file changes
    autoWatch: true,

    // start these browsers
    browsers: ['PhantomJS'],

    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    singleRun: false
  });
};
{%endhighlight%}

接下来我们用 [Bower](http://bower.io/) 来安装 AngularJS 和 angular-mocks。

### 用 Bower 安装 AngularJS 和 angular-mocks

注意我们在测试代码中通过 `angular-mocks.js` 指定了 [ngMock](https://docs.angularjs.org/api/ngMock)。ngMock 为 AngularJS 应用提供了很多测试工具方法，可以通过它来进行依赖注入和模块依赖管理，回头我们还会介绍到。AngularJS 和 angular-mocks 可以通过 Bower 来安装：

{%highlight bash%}
$ bower init # with default configuration
$ bower install angular --save
$ bower install angular-mocks --save-dev
{%endhighlight%}

`bower.json` 内容如下：

{%highlight js lineno%}
{
  "name": "angularjs-unit-testing",
  "version": "0.0.0",
  "authors": [
    "George Sun <sunwinner3@gmail.com>"
  ],
  "license": "MIT",
  "homepage": "http://www.codethoughts.info",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "angular": "~1.4.5"
  },
  "devDependencies": {
    "angular-mocks": "~1.4.5"
  }
}

{%endhighlight%}


关于 Mocha/Chai/Sinon，可以参考我的另一篇文章：[用 Mocha/Chai/Sinon 测试 JavaScript 代码](http://codethoughts.info/javascript/2015/07/18/javascript-bdd-with-mocha-chai-sinon/) ，这里不再赘述。

到此，我们的测试环境基本设置已经完成，下面我们来看看如何测试用 AngularJS 框架编写的各个组件。

## 测试 AngularJS 应用程序
AngularJS 框架鼓励开发者通过模块来组织代码，它通过 `module` 来定义和解析模块的依赖，可能是其他的模块。 同样 angular-mocks 也可以利用 `module(...)` 来查找所依赖的模块。比如：

{%highlight js lineno%}
beforeEach(module('myCustomModule'));
{%endhighlight%}

这段代码会在每个测试用例（通过 `it` 定义）运行之前查找所依赖的模块，如果所查找的模块不存在，那么这段代码会抛出异常。有了这个工具，我们通过实例来看看如何测试 AngularJS 应用程序的 Services, Controllers, Directives 等等。

### 测试 Service

Service 在 AngularJS 框架中泛指 Filter，Service，Factory 等几个子类，它们都可以看做面向对象设计中的单例模式。通常来说，Service 是比较易于测试的，因为 Service 所依赖的一般是其他 Service，它们可以通过依赖注入来管理和模拟（Mock），AngularJS 的设计使得注入其他 Service 工作变得异常简单，我们通过一个 Factory 实例来看看。

{%highlight js lineno%}
angular.module('factories', [])
.factory('factoryUT', ['$log', function($log) {
    "use strict";
    return {
        ook: function() {
            $log.warn('Ook.');
        }
    };
}]);
{%endhighlight%}

这里我们需要注入 `$log`，我也同样可以在测试代码中注入它：

{%highlight js lineno%}
describe('factories', function() {
    "use strict";

    beforeEach(module('factories'));

    var factoryUT;
    var $log;

    beforeEach(inject(function(_factoryUT_, _$log_) {
        factoryUT = _factoryUT_;
        $log = _$log_;
        sinon.stub($log, 'warn', function() {});
    }));

    describe('when invoked', function() {

        beforeEach(function() {
            factoryUT.ook();
        });

        it('should say Ook', function() {
            expect($log.warn.callCount).to.equal(1);
            expect($log.warn.args[0][0]).to.equal('Ook.');
        });
    });
});
{%endhighlight%}

这里特别需要注意的是 `_factoryUT_` 和 `_$log_`，它们都是 AngularJS 的一种使用惯例，通过下划线包装需要注入的模块，在依赖解析的时候，`angular-mocks` 会自动去除两端的下划线，详情可以参考 [Resolving References (Underscore Wrapping)](https://docs.angularjs.org/api/ngMock/function/angular.mock.inject)。

另外，我们使用 `sinon.stub()` 来为 `$log.warn()` 模拟函数的实现，AngularJS 通过依赖注入给我们带来的好处在这里得到了充分体现。

### 测试 Controller

AngularJS 是 MVC 架构，或者说是 MVW（Model-View-Whatever）架构。在 AngularJS 应用中，`controller` 被用作 Model 和 View 之间的代理。通过 controller，我们可以在上述两个组件之间传递状态和行为。下面我们通过一个在线文本编辑器实例来演示如何测试 AngularJS 的 controller 组件：

{%highlight js lineno%}
angular.module('textEditor', [])
.controller('EditionCtrl', ['$scope', function($scope) {
    "use strict";

    $scope.state = {toolbarVisible: true, documentSaved: true};
    $scope.document = {text: 'Some text'};

    $scope.$watch('document.text', function(value) {
        $scope.state.documentSaved = false;
    }, true);

    $scope.saveDocument = function() {
        $scope.sendHTTP($scope.document.text);
        $scope.state.documentSaved = true;
    };

    $scope.sendHTTP = function(content) {
        // payload creation, HTTP request, etc.
    };
}]);
{%endhighlight%}

上面代码中，`state` 状态可能被 View 层或者 Controller 代码修改，`$scope.$watch` 会通过监控状态变化来更新文档是否被保存的状态。另外， `toolbarVisible` 可能被 View 层的鼠标或者键盘事件所修改，这是我们在单元测试中所不能覆盖的，它需要 E2E 测试来覆盖。

测试代码如下：

{%highlight js lineno%}
describe('saving a document', function() {
    "use strict";

    var scope;
    var ctrl;

    beforeEach(module('textEditor'));

    beforeEach(inject(function($rootScope, $controller) {
        scope = $rootScope.$new();
        ctrl = $controller('EditionCtrl', {$scope: scope});
    }));

    it('should have an initial documentSaved state', function(){
        expect(scope.state.documentSaved).to.equal(true);
    });

    describe('documentSaved property', function() {
        beforeEach(function() {
            // We don't want extra HTTP requests to be sent
            // and that's not what we're testing here.
            sinon.stub(scope, 'sendHTTP', function() {});

            // A call to $apply() must be performed, otherwise the
            // scope's watchers won't be run through.
            scope.$apply(function () {
                scope.document.text += ' And some more text';
            });
        });

        it('should watch for document.text changes', function() {
            expect(scope.state.documentSaved).to.equal(false);
        });

        describe('when calling the saveDocument function', function() {
            beforeEach(function() {
                scope.saveDocument();
            });

            it('should be set to true again', function() {
                expect(scope.state.documentSaved).to.equal(true);
            });

            afterEach(function() {
                expect(scope.sendHTTP.callCount).to.equal(1);
                expect(scope.sendHTTP.args[0][0]).to.equal(scope.document.text);
            });
        });
    });
});
{%endhighlight%}

测试代码中比较有趣的地方是我们模拟了发送 HTTP 请求的方法，这样可以加速测试代码的运行速度，因为我们不需要发送真正的 HTTP 请求。在这里，请求的相应内容我们并不关心，我们只是每次返回同样的内容，并通过 `$scope.$apply()` 来触发一次文档保存动作。另外，我们也可以看到应该如何注入 `$scope` 和 `$controller` 依赖，并通过注入的 `$controller` 来查找我们所需要的 Controller。

关于 Controller 测试我们就介绍这么多，接下来看看如何测试 Directive。

### 测试 Directive

Directive 是 AngularJS 中最重要的概念。通过 Directive，我们可以通过模块化的代码来扩展 HTML，并在模块中封装行为和状态。Directive 通常是隔离的命名空间（isolated scope），所以在测试中我们可以把它当做一个黑盒子来看待，仅仅通过它暴露的接口还和它交互。下面我们通过一个实例来看看如何测试 Directive。

{%highlight js lineno%}
angular.module('myDirectives', [])
.directive('superButton', function() {
    "use strict";

    return {
        scope: {label: '=', callback: '&onClick'},
        replace: true,
        restrict: 'E',
        link: function(scope, element, attrs) {
            // do nothing.
        },
        template: '<div>' +
            '<div>{{label}}</div>' +
            '<button ng-click="callback()">Click me!</button>' +
            '</div>'
    };
});
{%endhighlight%}

上面的代码定义了一个按钮，我们可以通过 `$scope` 给它指定标签和行为，它们也是我们想要测试的地方。事实上，测试 Directive 的责任更多应该由 E2E 测试来承担，但是我们希望在单元测试中包括尽可能多的测试用例。这样我们可以尽快得到代码和行为是否正确的反馈。单元测试代码如下：

{%highlight js lineno%}
describe('directives', function() {
    "use strict";

    beforeEach(module('myDirectives'));

    var element;
    var outerScope;
    var innerScope;

    beforeEach(inject(function($rootScope, $compile) {
        element = angular.element('<super-button label="myLabel" on-click="myCallback()"></super-button>');

        outerScope = $rootScope;
        $compile(element)(outerScope);

        innerScope = element.isolateScope();

        outerScope.$digest();
    }));

    describe('label', function() {
        beforeEach(function() {
            outerScope.$apply(function() {
                outerScope.myLabel = "Hello world.";
            });
        });

        it('should be rendered', function() {
            expect(element[0].children[0].innerHTML).to.equal('Hello world.');
        });
    });

    describe('click callback', function() {
        var mySpy;

        beforeEach(function() {
            mySpy = sinon.spy();
            outerScope.$apply(function() {
                outerScope.myCallback = mySpy;
            });
        });

        describe('when the directive is clicked', function() {
            beforeEach(function() {
                var event = document.createEvent("MouseEvent");
                event.initMouseEvent("click", true, true);
                element[0].children[1].dispatchEvent(event);
            });

            it('should be called', function() {
                expect(mySpy.callCount).to.equal(1);
            });
        });
    });
});

{%endhighlight%}

上面代码正是测试了按钮的标签和行为。这里需要注意 `$compile` 的用法，`$compile` 可以把字符串或者 DOM 编译为模板，它返回一个模板函数，随后我们使用该函数绑定 `$scope` 和状态。详细的文档请参考 [AngularJS 官方文档](https://docs.angularjs.org/api/ng/service/$compile)。

另外，我们使用 DOM 源生的事件来触发按钮的 `onClick` 回调，从而把这个 Directive 当做一个黑盒子来测试。另外我们可以看到，在 Directive 内部，`myCallback` 被重命名为 `callback`，从 Directive 内部，我们只能通过 `callback` 来存取这个回调。原本我们可以像下面这样写测试代码：

{%highlight js lineno%}
    describe('click callback', function() {
        var mySpy;

        beforeEach(function() {
            mySpy = sinon.spy();
            innerScope.callback = mySpy;
        });

        describe('when the directive is clicked', function() {
            beforeEach(function() {
                var event = document.createEvent("MouseEvent");
                event.initMouseEvent("click", true, true);
                element[0].children[1].dispatchEvent(event);
            });

            it('should be called', function() {
                expect(mySpy.callCount).to.equal(1);
            });
        });
    });

{%endhighlight%}

上面这段代码仍然可以工作，可问题在于我们引用了 Directive 内部的状态，这样如果 Directive在内部把它的事件回调改为别的名字，相应的我们也要更新测试代码；另一个问题是这样一来，我们没有把 Directive 当成黑盒子来测试，我们必须要为我们事实上并不关心的内部状态编写测试代码。

### 测试 Provider

对 Provider 的测试相对复杂，我会在随后用一篇新的文章来解释 Provider 是什么，以及如何测试它。

### 测试 HTTP 请求

HTTP 是 Web 应用中不可或缺的一环，所以了解如何测试 HTTP 请求也很重要。在应用程序中，常见的 HTTP 请求有 `GET`, `POST`, `PUT`, `DELETE`。其中，`GET` 请求会向应用程序输入一些数据，而另外的三种请求则会向应用程序以外的第三方应用输入一些数据，这也是对于 HTTP 请求我们需要测试的地方。我们来看一个实例：

{%highlight js lineno%}
angular.module('HttpRequestExample', [])
.factory('httpReq', ['$http', function($http) {
    "use strict";
    return {
        sendMessage: function() {
            $http.get('http://it-ebooks-api.info/v1/search/JavaScript');
        }
    };
}]);

{%endhighlight%}

上面的代码像第三方服务发送了一个 GET 请求，我们需要在测试代码中 Mock 该 HTTP 请求，从而在单元测试中解除对第三方服务的依赖。测试代码如下：

{%highlight js lineno%}
describe('http', function() {
    "use strict";

    beforeEach(module('HttpRequestExample'));

    var httpReq;
    var $httpBackend;

    beforeEach(inject(function(_httpReq_, _$httpBackend_) {
        httpReq = _httpReq_;
        $httpBackend = _$httpBackend_;
    }));

    describe('when sending a message', function() {
        beforeEach(function() {
            $httpBackend.expectGET('http://it-ebooks-api.info/v1/search/JavaScript')
            .respond(200, {message: 'Ook.', id: 0});

            httpReq.sendMessage();
            $httpBackend.flush();
        });

        it('should send an HTTP GET request', function() {
            $httpBackend.verifyNoOutstandingExpectation();
            $httpBackend.verifyNoOutstandingRequest();
        });
    });
});

{%endhighlight%}

从测试代码中可以看出，`$httpBackend` 模拟了 HTTP 服务器的实现，并且定义了这个服务器需要响应的请求和响应值，这里我们定义了一个 GET 请求。另外，`$httpBackend.flush()` 的作用在于我们在真正响应请求之前可以配置响应的一些属性。这个模拟的 HTTP 服务器并不会立即响应请求，它会保持住当前请求，直到你明确的要求它返回响应值。

另外，`$httpBackend.verifyNoOutstandingExpectation` 和 `$httpBackend.verifyNoOutstandingRequest` 校验我们所定义的 HTTP 请求和响应都已经被触发，并给出了相应的响应，详细用法可以参见[angular-mocks 的文档](https://docs.angularjs.org/api/ngMock/service/$httpBackend)。

### ngMock 模块简介

ngMock 模块包含了一系列帮助我们测试 AngularJS 应用程序的工具方法，包括 $timeout, $interval, $log, $httpBackend, inject(...) 等等。其中的一部分我们在前面的单元测试实例中已经见过了。这里我还想再提一下 `module(...)` 和 `inject(...)` 两个函数。

[module(...)](https://docs.angularjs.org/api/ngMock/function/angular.mock.module) 用于在运行测试代码的时候查找和解析依赖，它的用法在前面的测试代码中也可以见到。而 [inject(...)](https://docs.angularjs.org/api/ngMock/function/angular.mock.inject) 用于创建一个 `$injector` 实例，用于解析依赖的引用。 

## 结论

AngularJS 的单元测试由于依赖注入的存在是比较容易编写的，这也是因为 AngularJS 在设计之初就把易测试性作为设计的首要目标之一。即便如此，AngularJS 也不能阻止我们作为应用程序开发者做傻事，我们要做的就是理解 AngularJS 的设计理念，做正确的事情。为了帮助我们开发者更好的写测试代码，AngularJS 开发者们为我们提供了 `angular-mocks` 类库，通过它，我们可以通过依赖注入来解析测试所需要的依赖，比较容易的用测试覆盖我们的代码，从而提高代码质量。

## 声明

本文部分实例代码来自 [An Introduction To Unit Testing In AngularJS Applications](http://www.smashingmagazine.com/2014/10/introduction-to-unit-testing-in-angularjs/)。


## 资源

[An Introduction To Unit Testing In AngularJS Applications](http://www.smashingmagazine.com/2014/10/introduction-to-unit-testing-in-angularjs/)

[Testing AngularJS Apps Using Karma](https://www.airpair.com/angularjs/posts/testing-angular-with-karma)

[AngularJS 主页](https://angularjs.org/)

[Karma 主页](http://karma-runner.github.io/0.8/index.html)

[Mocha 主页](http://mochajs.org/)

[Chai 主页](http://chaijs.com/)

[Sinon 主页](http://sinonjs.org/)

[Jasmine](https://github.com/jasmine/jasmine)

[Jasmine vs. Mocha, Chai, and Sinon](http://thejsguy.com/2015/01/12/jasmine-vs-mocha-chai-and-sinon.html)

[ngMock 文档](https://docs.angularjs.org/api/ngMock)