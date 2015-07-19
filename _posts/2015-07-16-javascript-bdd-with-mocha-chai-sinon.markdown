---
layout: post
title: 用 Mocha/Chai/Sinon 测试 JavaScript 代码
date: 2015-07-18T21:19:50+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: Mocha Chai Sinon Jasmine Mocha.js Chai.js Sinon.js Jasmine.js codethoughts.info codethoughts javascript 编程语言 JS  BDD TDD 测试 自动化 Grunt 
description: JavaScript Object Equality

---

## 引言

JavaScript 是一门动态语言，它没有类似静态语言的编译期语法检查，为了使得代码易于维护和重构，我们需要用自动化测试覆盖 JavaScript 代码。这篇文章将会介绍如何使用 Mocha 为 JavaScript 代码写自动化测试，主要是单元测试，因为单元测试是开发者日常工作中最常接触的自动化测试了。

本文将会将会使用 [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) 风格的测试代码，这种风格的测试写出的代码类似自然语言，常见的一些语法比如 `describe`, `expect`, `should` 等等。熟悉 Ruby 的人都应该知道 Ruby 社区大名鼎鼎的 [RSpec](http://rspec.info/)，在 JavaScript 社区也有类似的测试框架，比如我们今天要介绍的 [Mocha](http://mochajs.org/) 就是其中一种比较流行的 BDD 测试框架，利用它可以写出和 Spec 风格非常接近的测试代码，另外一个是 [Jasmine](https://github.com/jasmine/jasmine)，因为篇幅原因，我们今天不会涉及到。

## BDD vs. TDD
事实上，Mocha 既支持 TDD 风格的测试代码，也支持 BDD 风格的测试代码，我们来看两段分别演示了这两种风格的测试代码。

需要测试的Node.js代码：

{%highlight js lineno%}
// src/main.js
"use strict";

module.exports = function factorial(n) {
    if (n < 0) {
        return NaN;
    }
    if (n === 0) {
        return 1;
    }

    return n * factorial(n - 1);
};

{%endhighlight%}

这是TDD 风格的测试代码：

{%highlight js lineno%}
// test/tdd_main.js
"use strict";

var assert = require('assert'),
    factorial = require('../src/main');

suite('Test', function (){
    setup(function (){
        // Create any objects that we might need
    });

    suite('#factorial()', function (){
        test('equals 1 for sets of zero length', function (){
            assert.equal(1, factorial(0));
        });

        test('equals 1 for sets of length one', function (){
            assert.equal(1, factorial(1));
        });

        test('equals 2 for sets of length two', function (){
            assert.equal(2, factorial(2));
        });

        test('equals 6 for sets of length three', function (){
            assert.equal(6, factorial(3));
        });
    });
});

{%endhighlight%}

如何安装 Mocha 请参考本文下一节，安装完毕之后在项目根目录下运行 `mocha --ui tdd  test/tdd_main.js` 可以运行以上测试代码，这里需要指定测试代码为 `tdd`，否则会出现 `suite is not defined` 错误。

这是 BDD 风格的测试代码：

{%highlight js lineno%}
// test/bdd_main.js
"use strict";

var assert = require('assert'),
    should = require('chai').should(),
    factorial = require('../src/main');

describe('Test', function (){
    before(function(){
        // Stuff to do before the tests, like imports, what not
    });

    describe('#factorial()', function (){
        it('should return 1 when given 0', function (){
            factorial(0).should.equal(1);
        });

        it('should return 1 when given 1', function (){
            factorial(1).should.equal(1);
        });

        it('should return 2 when given 2', function (){
            factorial(2).should.equal(2);
        });

        it('should return 6 when given 3', function (){
            factorial(3).should.equal(6);
        });
    });

    after(function () {
        // 清理需要释放的资源
    });
});

{%endhighlight%}

在根目录下运行 `mocha --ui bdd  test/bdd_main.js` 可以运行以上测试代码，这里需要指定测试代码为 `bdd` 测试。

对 TDD 风格的测试在本文中的介绍到此为止，我们接下来主要介绍如何用 Mocha 及其配套库来为我们的 JavaScript 代码写 BDD 风格的测试代码。Mocha 常见的配套库有[Chai](http://chaijs.com/) 和 [Sinon](http://sinonjs.org/)，首先我们来看看它们都是什么。

## 准备 Mocha 测试环境

[Mocha](http://mochajs.org/) 是一个 JavaScript 测试框架，可以用来运行测试代码，它没有内置的 Assertion，Mock 和 Stub 功能。一般我们用 [Chai](http://chaijs.com/) 来为它提供断言，用 [Sinon](http://sinonjs.org/) 为它提供 Mock 和 Stub 功能。Mocha 可以用来测试 Node.js 和 浏览器的 JavaScript 代码。本文中的样例代码都是 Node.js 代码，如果你需要用它来测试运行在浏览器中的 JavaScript 代码，可以参考它的主页去设置测试环境。

Mocha 用于运行测试代码，我们需要全局安装：
`npm install -g mocha`

安装 Chai：
`npm install -D chai`， -D 等同于 --save-dev，参见[npm configuration](https://docs.npmjs.com/misc/config)

安装 Sinon：
`npm install -D sinon`


## 需要测试的代码

为了全面的展示 Mocha 和 Jasmine 这两个测试框架，我们这里用它来测试一段比较复杂的代码。这段代码是用 JavaScript 实现了 [有向图](https://en.wikipedia.org/wiki/Directed_graph)，并且图的边可以带有权重。如果你对这段代码有兴趣，可以研究一下具体的实现，代码也有详细的注释；而如果你仅仅对如何写测试有兴趣，那么可以忽略具体的实现，把它仅仅当成黑盒子。在阅读代码的时候请注意，我们这里所写的是 Node.js 代码，它用 CommonJS 来管理依赖。简单讲，就是用 `module.exports` 来导出公开的接口，没有指定导出的函数很变量无法被其他源文件的代码引用，它们的范围就会被局限在自身的源文件中；如果需要应用其他源文件的依赖，则需要 `require` 来指定需要导入的依赖。除了 [有向图](https://en.wikipedia.org/wiki/Directed_graph)，我还用 JavaScript 实现了其他常见数据结构和算法，并且都有测试覆盖，如果有兴趣，可以参考我的 GitHub Repo: [Data Structures and Algorithms with JavaScript](https://github.com/puffsun/js_datastructures_algorithms)，里面有详细的指示如何去运行这个 Repo 里的测试代码，所有的测试都使用 Mocha 框架编写。

{%highlight js lineno%}
/*
Graph implemented as a modified incidence list. O(1) for every typical
operation except `removeNode()` at O(E) where E is the number of edges.

## Overview example:

```js
var graph = new Graph;
graph.addNode('A'); // => a node object. For more info, log the output or check
                    // the documentation for addNode
graph.addNode('B');
graph.addNode('C');
graph.addEdge('A', 'C'); // => an edge object
graph.addEdge('A', 'B');
graph.getEdge('B', 'A'); // => undefined. Directed edge!
graph.getEdge('A', 'B'); // => the edge object previously added
graph.getEdge('A', 'B').weight = 2 // weight is the only built-in handy property
                                   // of an edge object. Feel free to attach
                                   // other properties
graph.getInEdgesOf('B'); // => array of edge objects, in this case only one;
                         // connecting A to B
graph.getOutEdgesOf('A'); // => array of edge objects, one to B and one to C
graph.getAllEdgesOf('A'); // => all the in and out edges. Edge directed toward
                          // the node itself are only counted once
forEachNode(function(nodeObject) {
  console.log(node);
});
forEachEdge(function(edgeObject) {
  console.log(edgeObject);
});
graph.removeNode('C'); // => 'C'. The edge between A and C also removed
graph.removeEdge('A', 'B'); // => the edge object removed
```

## Properties:

- nodeSize: total number of nodes.
- edgeSize: total number of edges.
 */

"use strict";

var hasProp = {}.hasOwnProperty;

function Graph() {
    this._nodes = {};
    this.nodeSize = 0;
    this.edgeSize = 0;
}

Graph.prototype.addNode = function(id) {

    /*
    The `id` is a unique identifier for the node, and should **not** change
    after it's added. It will be used for adding, retrieving and deleting
    related edges too.

    **Note** that, internally, the ids are kept in an object. JavaScript's
    object hashes the id `'2'` and `2` to the same key, so please stick to a
    simple id data type such as number or string.

    _Returns:_ the node object. Feel free to attach additional custom properties
    on it for graph algorithms' needs. **Undefined if node id already exists**,
    as to avoid accidental overrides.
     */
    if (!this._nodes[id]) {
        this.nodeSize++;
        this._nodes[id] = {
            _outEdges: {},
            _inEdges: {}
        };
        return this._nodes[id];
    }
};

Graph.prototype.getNode = function(id) {

    /*
    _Returns:_ the node object. Feel free to attach additional custom properties
    on it for graph algorithms' needs.
     */
    return this._nodes[id];
};

Graph.prototype.removeNode = function(id) {

    /*
    _Returns:_ the node object removed, or undefined if it didn't exist in the
    first place.
     */
    var nodeToRemove = this._nodes[id],
        outEdgeId, inEdgeId;

    if (!nodeToRemove) {
        return;
    } else {
        for (outEdgeId in nodeToRemove._outEdges) {
            if (!hasProp.call(nodeToRemove._outEdges, outEdgeId)) {
                continue;
            }
            this.removeEdge(id, outEdgeId);
        }
        for (inEdgeId in nodeToRemove._inEdges) {
            if (!hasProp.call(nodeToRemove._inEdges, inEdgeId)) {
                continue;
            }
            this.removeEdge(inEdgeId, id);
        }
        this.nodeSize--;
        delete this._nodes[id];
    }
    return nodeToRemove;
};

Graph.prototype.addEdge = function(fromId, toId, weight) {
    var edgeToAdd, fromNode, toNode;
    if (!weight) {
        weight = 1;
    }

    /*
    `fromId` and `toId` are the node id specified when it was created using
    `addNode()`. `weight` is optional and defaults to 1. Ignoring it effectively
    makes this an unweighted graph. Under the hood, `weight` is just a normal
    property of the edge object.

    _Returns:_ the edge object created. Feel free to attach additional custom
    properties on it for graph algorithms' needs. **Or undefined** if the nodes
    of id `fromId` or `toId` aren't found, or if an edge already exists between
    the two nodes.
     */
    if (this.getEdge(fromId, toId)) {
        return;
    }
    fromNode = this._nodes[fromId];
    toNode = this._nodes[toId];
    if (!fromNode || !toNode) {
        return;
    }
    edgeToAdd = {
        weight: weight
    };
    fromNode._outEdges[toId] = edgeToAdd;
    toNode._inEdges[fromId] = edgeToAdd;
    this.edgeSize++;
    return edgeToAdd;
};

Graph.prototype.getEdge = function(fromId, toId) {

    /*
    _Returns:_ the edge object, or undefined if the nodes of id `fromId` or
    `toId` aren't found.
     */
    var fromNode, toNode;
    fromNode = this._nodes[fromId];
    toNode = this._nodes[toId];
    if (fromNode && toNode) {
        return fromNode._outEdges[toId];
    }
};

Graph.prototype.removeEdge = function(fromId, toId) {

    /*
    _Returns:_ the edge object removed, or undefined of edge wasn't found.
     */
    var edgeToDelete, fromNode, toNode;
    fromNode = this._nodes[fromId];
    toNode = this._nodes[toId];
    edgeToDelete = this.getEdge(fromId, toId);
    if (!edgeToDelete) {
        return;
    }
    delete fromNode._outEdges[toId];
    delete toNode._inEdges[fromId];
    this.edgeSize--;
    return edgeToDelete;
};

Graph.prototype.getInEdgesOf = function(nodeId) {

    /*
    _Returns:_ an array of edge objects that are directed toward the node, or
    empty array if no such edge or node exists.
     */
    var fromId, inEdges, ref, toNode;
    toNode = this._nodes[nodeId];
    inEdges = [];
    if (toNode) {
        ref = toNode._inEdges;
    } else {
        // same as undefined
        ref = void 0;
    }
    for (fromId in ref) {
        if (!hasProp.call(ref, fromId)) {
            continue;
        }
        inEdges.push(this.getEdge(fromId, nodeId));
    }
    return inEdges;
};

Graph.prototype.getOutEdgesOf = function(nodeId) {

    /*
    _Returns:_ an array of edge objects that go out of the node, or empty array
    if no such edge or node exists.
     */
    var fromNode, outEdges, ref, toId;
    fromNode = this._nodes[nodeId];
    outEdges = [];

    if (fromNode) {
        ref = fromNode._outEdges;
    } else {
        ref = 0;
    }
    for (toId in ref) {
        if (!hasProp.call(ref, toId)) {
            continue;
        }
        outEdges.push(this.getEdge(nodeId, toId));
    }
    return outEdges;
};

Graph.prototype.getAllEdgesOf = function(nodeId) {

    /*
    **Note:** not the same as concatenating `getInEdgesOf()` and
    `getOutEdgesOf()`. Some nodes might have an edge pointing toward itself.
    This method solves that duplication.

    _Returns:_ an array of edge objects linked to the node, no matter if they're
    outgoing or coming. Duplicate edge created by self-pointing nodes are
    removed. Only one copy stays. Empty array if node has no edge.
     */
    var inEdges = this.getInEdgesOf(nodeId),
        outEdges = this.getOutEdgesOf(nodeId),
        selfEdge = this.getEdge(nodeId, nodeId),
        i, len;

    if (inEdges.length === 0) {
        return outEdges;
    }
    for (i = 0, len= inEdges.length; i < len; i++) {
        if (inEdges[i] === selfEdge) {
            swap(inEdges[inEdges.length - 1], inEdges[i]);
            inEdges.pop();
            break;
        }
    }
    return inEdges.concat(outEdges);
};

function swap(a, b) {
    var tmp = a;
    a = b;
    b = tmp;
}

Graph.prototype.forEachNode = function(operation) {

    /*
    Traverse through the graph in an arbitrary manner, visiting each node once.
    Pass a function of the form `fn(nodeObject, nodeId)`.

    _Returns:_ undefined.
     */
    var nodeId, nodeObject;
    for (nodeId in this._nodes) {
        if (!hasProp.call(this._nodes, nodeId)) {
            continue;
        }
        nodeObject = this._nodes[nodeId];
        operation(nodeObject, nodeId);
    }
};

Graph.prototype.forEachEdge = function(operation) {

    /*
    Traverse through the graph in an arbitrary manner, visiting each edge once.
    Pass a function of the form `fn(edgeObject)`.

    _Returns:_ undefined.
     */
    var edgeObject, nodeId, nodeObject, toId;
    for (nodeId in this._nodes) {
        if (!hasProp.call(this._nodes, nodeId)) {
            continue;
        }
        nodeObject = this._nodes[nodeId];
        for (toId in nodeObject._outEdges) {
            if (!hasProp.call(nodeObject._outEdges, toId)) {
                continue;
            }
            edgeObject = nodeObject._outEdges[toId];
            operation(edgeObject);
        }
    }
};

module.exports = Graph;

{%endhighlight%}

## 测试代码

下面我们来看看测试代码，我们这里不会全面覆盖上面需要测试的源码，如果希望查看完整的测试源码，请参见[puffsun/js_datastructures_algorithms](https://github.com/puffsun/js_datastructures_algorithms/tree/master/test/datastructures/graph)。

首先引入依赖，并声明测试上下文（Context）和初始化代码：

{%highlight js lineno%}
"use strict";

var Graph = require("../../../src/datastructures/graph/graph"),
    expect = require('chai').expect,
    sinon = require("sinon");


describe("Test Graph", function() {
    before(function () {
        console.log("top before");
    });
    after(function () {
        console.log("top after");
    });
    beforeEach(function () {
        console.log("top beforeEach");
    });
    afterEach(function () {
        console.log("top afterEach");
    });
}
{%endhighlight%}

上面的代码中我们可以看到函数 `before(...)`，`after(...)`，`beforeEach(...)`，`afterEach(...)`，他们的区别是 `before` 和 `after` 仅仅会在所有测试运行之前和之后运行一次，而另外两个则会在每一个测试用例运行之前都被调用，他们用于初始化和清理一些资源。

接下来我们来看看如何测试新增图节点的函数 `addNode(...)`，这里的代码是嵌套在上面的 `describe("Test Graph", function() {}` 中间的，测试代码如下：

{%highlight js lineno%}
describe("Add node", function() {
    var graph = new Graph();
    it("should have 0 edge and 0 node initially", function() {
        expect(graph.nodeSize).to.equal(0);
        expect(graph.edgeSize).to.equal(0);
    });
    it("should return the node object added, or undefined if the id exists", function() {
        expect((graph.addNode("item")) instanceof Object).to.equal(true);
        expect((graph.addNode("1")) instanceof Object).to.equal(true);
        expect((graph.addNode(null)) instanceof Object).to.equal(true);
    });
    it("should return undefined if the node id already exists", function() {
        expect(graph.addNode("item")).to.equal(undefined);
        expect(graph.addNode("1")).to.equal(undefined);
        expect(graph.addNode(null)).to.equal(undefined);
    });
    it("should have kept the node size constant with non-insertions", function() {
        expect(graph.nodeSize).to.equal(3);
    });
});

{%endhighlight%}

Mocha 通过 Chai 这个断言库可以支持 `assert`，`should` 和 `expect` 等多种断言方式，我们在前面已经见过了 `should` 方式的断言，这里我们使用 `expect` 风格的断言，基本语法为 `expect(...).to.equal()`。如果是数组我们就需要 Chai 提供的另一个比较函数`eql(...)`，来看另一段测试代码体会一下它们的区别，这段代码测试获取有向图中指向某节点的所有边：

{%highlight js lineno%}
    describe("Get all in edges", function() {
        var graph = new Graph(),
            graph2 = new Graph();

        it("should return empty array for a non-existant node", function() {
            expect(graph.getOutEdgesOf("6")).to.eql([]);
            expect(graph.getOutEdgesOf(void 0)).to.eql([]);
        });

        it("should return empty array for no edges", function() {
            addNodesTo(graph);
            expect(graph.getInEdgesOf("1")).to.eql([]);
            expect(graph.getInEdgesOf("2")).to.eql([]);
            expect(graph.getInEdgesOf("6")).to.eql([]);
        });

        it("should return the in edges", function() {
            addNodesTo(graph2, true);

            /*
            1 <- 2 <-> 3
            |^   ^     ^
            v \  |     |
            4   \5     6 <->
             */
            expect(graph2.getInEdgesOf("1").length).to.equal(2);
            expect(graph2.getInEdgesOf("1")).to.contain(graph2.getEdge("2", "1"));
            expect(graph2.getInEdgesOf("1")).to.contain(graph2.getEdge("5", "1"));
            expect(graph2.getInEdgesOf("2").length).to.equal(2);
            expect(graph2.getInEdgesOf("2")).to.contain(graph2.getEdge("3", "2"));
            expect(graph2.getInEdgesOf("2")).to.contain(graph2.getEdge("5", "2"));
            expect(graph2.getInEdgesOf("3").length).to.equal(2);
            expect(graph2.getInEdgesOf("3")).to.contain(graph2.getEdge("2", "3"));
            expect(graph2.getInEdgesOf("3")).to.contain(graph2.getEdge("6", "3"));
            expect(graph2.getInEdgesOf("4").length).to.equal(1);
            expect(graph2.getInEdgesOf("4")).to.contain(graph2.getEdge("1", "4"));
            expect(graph2.getInEdgesOf("5")).to.eql([]);
            expect(graph2.getInEdgesOf("6").length).to.equal(1);
            expect(graph2.getInEdgesOf("6")).to.contain(graph2.getEdge("6", "6"));
        });
    });

{%endhighlight%}

我们可以看到 `eql` 比较的是数组的元素内容，只要两个数组中元素相同，且顺序相同，`eql` 函数就认为它们是相同的，而 `equal` 函数则不然，它比较的是对象在内存中的地址，两个对象必须指向同一个对象实例才认为他们是相同的，比如：

{%highlight js lineno%}
            expect([1,2]).to.eql([1,2]);     // true
            expect([1,2]).to.equal([1,2]);   // false
{%endhighlight%}

对于 JavaScript 对象相等性的比较细节请参考我的上一篇博客 [判定 JavaScript 对象是否相等](http://codethoughts.info/javascript/2015/07/12/javascript-object-equality/)。

`addNodesTo(...)` 是一个测试辅助函数，定义如下：
{%highlight js lineno%}
function addNodesTo(graph, addEdges) {
    var initEdgeSize, initNodeSize;
    if (addEdges === null) {
        addEdges = false;
    }

    initNodeSize = graph.nodeSize;
    graph.addNode("1");
    graph.addNode("2");
    graph.addNode("3");
    graph.addNode("4");
    graph.addNode("5");
    graph.addNode("6");
    expect(graph.nodeSize).to.equal(initNodeSize + 6);
    if (addEdges) {

        /*
        1 <- 2 <-> 3
        |^   ^     ^
        v \  |     |
        4   \5     6 <->
         */
        initEdgeSize = graph.edgeSize;
        expect(initEdgeSize).to.equal(0);
        graph.addEdge("1", "4", 9);
        graph.addEdge("2", "1", 9);
        graph.addEdge("2", "3", 9);
        graph.addEdge("3", "2", 9);
        graph.addEdge("5", "1", 9);
        graph.addEdge("5", "2", 9);
        graph.addEdge("6", "3", 9);
        graph.addEdge("6", "6", 9);
        expect(graph.edgeSize).to.equal(initEdgeSize + 8);
    }
}

{%endhighlight%}

如果需要测试函数是否被调用，或者调用的次数是否符合预期，我们需要对象的 Mock 和 Stub 功能，这里就去要 [Sinon.js](http://sinonjs.org/) 的支持了，来看一段测试代码：

{%highlight js lineno%}
describe("Traverse through each node", function() {
    var graph = new Graph();

    it("shouldn't call the callback for an empty graph", function() {
        var callback = sinon.spy();

        graph.forEachNode(callback);
        sinon.assert.notCalled(callback);
    });

    it("should reach each node once", function() {
        var callback = sinon.spy();
        addNodesTo(graph);
        graph.forEachNode(callback);
        expect(callback.callCount).to.equal(6);
    });

    it("should pass nodeObject and nodeId to the callback", function() {
        var callback = sinon.spy();
        graph.forEachNode(callback);
        expect(callback.lastCall.args.length).to.equal(2);
        expect(callback.lastCall.args[0] instanceof Object).to.equal(true);
        expect(callback.lastCall.args[1]).to.equal("6");
    });
});
{%endhighlight%}

这里可以看到，我们可以通过 `var callback = sinon.spy();` 来创建函数的 Stub，这里被称为 `Spy`，通过 `sinon.assert.notCalled` 来测试函数是否已经被调用。另外，我们还可以通过 `callback.lastCall` 来获取最后一次调用的函数，并通过 `args` 来取得函数的入参。这里只是展示了 Sinon.js 一小部分功能，更多细节请参考 [Sinon.js 的文档](http://sinonjs.org/docs/)。

完整的测试代码和 Gruntfile.js 构建脚本请参见[GitHub Repo - graph_spec.js](https://github.com/puffsun/js_datastructures_algorithms/blob/master/test/datastructures/graph/graph_spec.js) 和 [Gruntfile.js](https://github.com/puffsun/js_datastructures_algorithms/blob/master/Gruntfile.js)，Repo 的 README 文件还详细介绍了如何运行所有的测试代码。

另外一个比较流行的 JavaScript  BDD测试框架是[Jasmine](https://github.com/jasmine/jasmine)。它的思路和 Mocha 不同，是一个完备的测试框架，内置了断言库，Mock/Stub 等高级功能，如果你喜欢一栈式的解决方案，可以考虑使用它。事实上，Mocha 可以写出和 Jasmine 风格非常相似的测试代码，只是 Mocha 需要和其他的库比如 Chai 和 Sinon 配合使用，而 Jasmine 是一栈式方案，可以根据你的喜好来选择，我个人更喜欢 Mocha 带来的灵活性。

## 资源

[Data Structures and Algorithms with JavaScript](https://github.com/puffsun/js_datastructures_algorithms)

[Mocha 主页](http://mochajs.org/)

[Chai 主页](http://chaijs.com/)

[Sinon 主页](http://sinonjs.org/)

[Jasmine](https://github.com/jasmine/jasmine)

[Jasmine vs. Mocha, Chai, and Sinon](http://thejsguy.com/2015/01/12/jasmine-vs-mocha-chai-and-sinon.html)
