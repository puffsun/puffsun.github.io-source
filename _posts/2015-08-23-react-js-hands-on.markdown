---

layout: post
title: React.js 初体验 
date: 2015-08-23T09:18:44+08:00
author: George Sun
comments: true
categories: ["javascript"]
keywords: React React.js Facebook 框架 codethoughts.info codethoughts javascript 编程语言 JS 
description: React.js Hands on

---
目前最火的 JavaScript 类库非 React.js 莫属，这几天花了点时间试玩，这篇就是一些心得体会，另外会通过一个 Hello world 项目来讲解 React.js 的一些概念和技术。

## 简介

从 [React.js 首页](http://facebook.github.io/react/) 的简介中可以看出，React应该被用作 MVC 的 View 层。React 把 View 层看做由一个或者多个组件组成，就像搭积木一样。在开发 React 应用的时候，你应该把你期望应用所拥有的 View 分割成单个的组件(Component)，并用这些组件来组合出一个完整的 View。React 另外一个比较重要的概念是虚拟 DOM (Virtual DOM)，这里我暂时不再赘述，毕竟这只是一篇入门的文章，感兴趣的读者可以参考 [The Secrets of React’s virtual DOM](http://fluentconf.com/fluent2014/public/schedule/detail/32395) 和 [React’s diff algorithm](http://calendar.perfplanet.com/2013/diff/)。我们首先来设置 React.js 开发环境并开发一个React 版的 Hello world。

## 设置环境

首先我们需要下载 React 库。访问 [http://reactjs.com](http://reactjs.com)，你会被重定向到[http://facebook.github.io/react/](http://facebook.github.io/react/)。点击 [Download React v0.13.3](http://facebook.github.io/react/downloads.html)，在打开的新页面上点击 [Download Starter Kit 0.13.3](http://facebook.github.io/react/downloads/react-0.13.3.zip)。下载完成之后把这个 zip 包解压到某个目录下，比如：

{%highlight bash%}
mkdir ~/source/react-startkit
mv ~/Downloads/react-0.13.3/ ~/source/react-startkit/react
cd ~/source/react-startkit/react
{%endhighlight%}

好了，基本设置到此为止，接下来该看实际的代码了。首先我会用 JavaScript 开发这个应用，文章的后半部分我会用 JSX 重写该 Hello world， 并介绍一些配套的工具和类库。

## Hello React

新建一个源文件 `~/source/react-startkit/react/helloworld.html`：
{%highlight js lineno%}
<!DOCTYPE html>
<html>
  <head>
    <title>hello React</title>
    <meta charset="utf-8">
  </head>
  <body>
    <div id="app">
    <!-- React 应用在这里渲染 -->
    </div>
    <script src="react/build/react.js"></script>
    <script>
    //  JavaScript 代码在这里
    </script>
  </body>
</html>
{%endhighlight%}

上面的代码没有需要解释的地方，接下来我们看如何利用引入的 React.js 来实现 Hello world 应用。在 `<script>` 标签内键入如下代码：

{%highlight js lineno%}
React.render(    React.DOM.h1(null, "Hello world!"), document.getElementById("app"));
{%endhighlight%}

刷新一下浏览器，一个崭新的 React hello world 就成功运行了。以 Chrome 浏览器为例，如果你在 `Hello world` 上右击并选择 `Inspect Element`，你会在 Chrome Dev Tools 里看到 `<div>` 标签内被填入了如下内容：

{%highlight html%}
<div id="app">
    <h1 data-reactid=".0">Hello world!</h1>
</div>
{%endhighlight%}

上面的 `<h1>` 标签就是 React.js 生成的内容。事实上，你可以通过 Chrome Developer Tools 达到同样的目的，比如：
{%highlight js lineno%}
var h = React.render(    React.DOM.h1(null, "Hello world!"), document.getElementById("app"));
console.log(h.getDOMNode()); // <h1 data-reactid=".0">Hello world!</h1>
{%endhighlight%}

下面我们来看看上面代码中需要注意的几个地方。

## 近距离看 React

首先，我们注意到了引入 React.js 以后，我们的应用中存在一个全局变量 `React`，React 提供的 API 都是通过它来暴露的。所幸它所提供的 API 数量不算多，也算是帮助我们这些苦逼的程序员们节约了一点有限的脑细胞。我们可以通过 Chrome Developer Console 来探索一下：

{%highlight js%}
console.log(React);
// 输出：
    Children: Object
    Component: ReactComponent(props, context)
    DOM: Object
    PropTypes: Object
    __spread: assign(target, sources)
    cloneElement: (element, props, children)
    constructAndRenderComponent: (constructor, props, container)
    constructAndRenderComponentByID: (constructor, props, id)
    createClass: (spec)
    createElement: (type, props, children)
    createFactory: (type)
    createMixin: (mixin)
    findDOMNode: findDOMNode(componentOrElement)
    initializeTouchEvents: (shouldUseTouch)
    isValidElement: (object)
    render: ()
    renderToStaticMarkup: renderToStaticMarkup(element)
    renderToString: renderToString(element)
    unmountComponentAtNode: (container)
    version: "0.13.3"
    withContext: (newContext, scopedCallback)
{%endhighlight%}

在上面的代码中，我们见到了其中的 `render()`, `DOM`, `DOM.h1()` 这几个 API，目前我们先只关心它们。

其次，上面的代码已经体现了 React.js 中组件的概念。在开发 React.js 应用的时候，我们通过 React 提供的 API 创建组件，并通过需要的方式来组合组件，从而得到我们需要的应用。在我们的 Hello world 应用中，我们创建了一个自定义组件，虽然特别简单，但它仍然是一个独立的组件。它是通过 `React.DOM` 所提供的接口方法，叫做 `React.DOM.h1()`，来创建。事实上 `React.DOM` 提供了很多类似的接口方法，我们可以通过 Chrome Developer Tools 来看看：

{%highlight js lineno%}
console.log(Object.keys(React.DOM).join(","))

// 输出： a,abbr,address,area,article,aside,audio,b,base,bdi,bdo,
big,blockquote,body,br,button,canvas,caption,cite,code,col,
colgroup,data,datalist,dd,del,details,dfn,dialog,div,dl,dt,em,
embed,fieldset,figcaption,figure,footer,form,h1,h2,h3,h4,h5,h6,
head,header,hr,html,i,iframe,img,input,ins,kbd,keygen,label,
legend,li,link,main,map,mark,menu,menuitem,meta,meter,nav,
noscript,object,ol,optgroup,option,output,p,param,picture,
pre,progress,q,rp,rt,ruby,s,samp,script,section,select,
small,source,span,strong,style,sub,summary,sup,table,
tbody,td,textarea,tfoot,th,thead,time,title,tr,track,
u,ul,var,video,wbr,circle,clipPath,defs,ellipse,g,line,
linearGradient,mask,path,pattern,polygon,polyline,
radialGradient,rect,stop,svg,text,tspan
{%endhighlight%}

这些都是对 `React.createElement(...)` 方法的封装，用来方便我们来创建自定义组件。

最后，我们看到了 `document.getElementById("app")`，它告诉 React.js 在应用的哪个 DOM 节点渲染我们的自定义组件。

## JSX

上面我们是通过纯粹的 JavaScript 来创建 React.js 自定义组件，对于我们的 Hello world 应用它还可以保证代码的可维护性，一旦组件比较复杂，并有多层嵌套的时候，代码很快就失去控制了，比如下面的代码：

{%highlight js lineno%}
React.render(
	React.DOM.h1({
		id: "my-heading"
	}, 
	React.DOM.span(null,
		React.DOM.em(null, "Hell"),
		"o"
	),	" world!"),
    document.getElementById('app')
);
{%endhighlight%}

如果我们使用 JSX，则代码可以像这样写：

{%highlight js lineno%}
React.render(
	<h1 id="my-heading">
    	<span><em>Hell</em>o</span> world!
	</h1>,
    document.getElementById('app')
);
{%endhighlight%}

我们可以看到上面的代码很像 HTML，它是 React.js 所提供的特殊语法，可以用来简化自定义组件的编写。问题是它不是合法的 JavaScript 代码，那如何才能运行它呢？我们需要一个转译器(Transpiler)来预处理它。当然，你也想到了，React.js 也提供了对应的工具，也可以通过第三方工具来做。为了让文章不再拗口，我下面将会直接使用 Transpile 和 Polyfill 这两个术语不加翻译。

## Transpile vs. Polyfill

Polyfill 对于前端工程师来说一定不会陌生。由于 JavaScript 社区的爆发性增长，也由于 ES 标准在浏览器中的实现总是滞后于 ES 标准推出的时间。那么对于一些新特性，比如 `Array.map` 就需要 Polyfill 类库来实现，比如：

{%highlight js lineno%}
if (!Array.prototype.map) {
  Array.prototype.map = function() {
  // implement the method
  };
}
{%endhighlight%}

那对于一些新的语法，比如 ES2015 的 `class`，没法用 Polyfill 来实现，就需要一些 Transpiler 来对代码进行预处理。Transpile 过程其实就是把新语法翻译成浏览器已经支持的语法的过程。其实也不难理解，类似 CoffeeScript 代码被翻译成 JavaScript 的过程。

## JSX Transpile

可以有几种办法来转译 JSX，这篇文章里我要介绍的是分别是从客户端，通过 react-tools，和通过 Babel 来转译。

### 客户端 Transpile

[React Starter Kit](https://facebook.github.io/react/downloads/react-0.13.3.zip) 提供了 `JSXTransformer.js`，我们可以通过它实现在浏览器中 Transpile JSX：

{%highlight html%}
<script src="/path/to/react.js"></script><script src="/path/to/JSXTransformer.js"></script>
<script type="text/jsx"> React.render(/*...*/);</script>
{%endhighlight%}

这种办法的优点是你可以快速使用 React.js，并充分利用 JSX 的语法来简化组件的开发；缺点是性能会比较差。因为脚本加载以后，在运行之前，`JSXTransformer.js` 要首先把 JSX 转换成 JavaScript 才可以在浏览器中运行。

### React-tools

任何一个正式的项目都会有自己的标准构建流程，我们要做的就是把 JSX 转译过程加入到这个构建流程中。首先我们需要安装 react-tools:

{%highlight bash%}
$ npm install -g react-tools
$ jsx -h

  Usage: jsx [options] <source directory> <output directory> [<module ID> [<module ID> ...]]

  Options:

    -h, --help                               output usage information
    -V, --version                            output the version number
    -c, --config [file]                      JSON configuration file (no file or - means STDIN)
    -w, --watch                              Continually rebuild
    -x, --extension <js | coffee | ...>      File extension to assume when resolving module identifiers
    --relativize                             Rewrite all module identifiers to be relative
    --follow-requires                        Scan modules for required dependencies
    --use-provides-module                    Respect @providesModules pragma in files
    --cache-dir <directory>                  Alternate directory to use for disk cache
    --no-cache-dir                           Disable the disk cache
    --source-charset <utf8 | win1252 | ...>  Charset of source (default: utf8)
    --output-charset <utf8 | win1252 | ...>  Charset of output (default: utf8)
    --harmony                                Turns on JS transformations such as ES6 Classes etc.
    --target [version]                       Specify your target version of ECMAScript. Valid values are "es3" and "es5". The default is "es5". "es3" will avoid uses of defineProperty and will quote reserved words. WARNING: "es5" is not properly supported, even with the use of es5shim, es5sham. If you need to support IE8, use "es3".
    --strip-types                            Strips out type annotations.
    --es6module                              Parses the file as a valid ES6 module. (Note that this means implicit strict mode)
    --non-strict-es6module                   Parses the file as an ES6 module, except disables implicit strict-mode. (This is useful if you're porting non-ES6 modules to ES6, but haven't yet verified that they are strict-mode safe yet)
    --source-map-inline                      Embed inline sourcemap in transformed source

$ jsx --watch source/ build/
{%endhighlight%}

通过 react-tools 可以把 JSX 代码转译成 JavaScript 代码，从而浏览器可以理解并运行。

### Babel

Babel 是一个开源项目，它支持 JSX 转换为 JavaScript 代码的功能，但也包括了其他的功能，包括吧 ES2015 标准的语法转译为目前浏览器支持的语法。详细的项目介绍可以参考 [Babel 主页](https://babeljs.io/)。事实上从 React.js v0.14 开始，`JSXTransformer.js` 将不再包括在 React.js 的安装包中， 它也不再是默认的 JSX Transpiler，React.js 将默认使用 [Babel](https://babeljs.io/) 来实现JSX转译。

{%highlight bash%}
$ npm install --global babel
$ babel -h

  Usage: babel [options] <files ...>

  Options:

    -h, --help                           output usage information
    -f, --filename [filename]            filename to use when reading from stdin - this will be used in source-maps, errors etc
    --module-id [string]                 specify a custom name for module ids
    --retain-lines                       retain line numbers - will result in really ugly code
    --no-non-standard                    enable/disable support for JSX and Flow (on by default)
    --experimental                       allow use of experimental transformers
    --no-highlight-code                  enable/disable ANSI syntax highlighting of code frames (on by default)
    -e, --stage [number]                 ECMAScript proposal stage version to allow [0-4]
    -b, --blacklist [transformerList]    blacklist of transformers to NOT use
    -l, --whitelist [transformerList]    whitelist of transformers to ONLY use
    --optional [transformerList]         list of optional transformers to enable
    -m, --modules [string]               module formatter type to use [common]
    -M, --module-ids                     insert an explicit id for modules
    -L, --loose [transformerList]        list of transformers to enable loose mode ON
    -P, --jsx-pragma [string]            custom pragma to use with JSX (same functionality as @jsx comments)
    --plugins [list]
    --ignore [list]                      list of glob paths to **not** compile
    --only [list]                        list of glob paths to **only** compile
    --no-comments                        strip/output comments in generated output (on by default)
    --compact [booleanString]            do not include superfluous whitespace characters and line terminators [true|false|auto]
    -k, --keep-module-id-extensions      keep extensions when generating module ids
    -a, --auxiliary-comment [string]     [DEPRECATED] renamed to auxiliaryCommentBefore
    --auxiliary-comment-before [string]  attach a comment before all helper declarations and auxiliary code
    --auxiliary-comment-after [string]   attach a comment after all helper declarations and auxiliary code
    -r, --external-helpers               uses a reference to `babelHelpers` instead of placing helpers at the top of your code.
    -s, --source-maps [booleanString]    [true|false|inline]
    --source-map-name [string]           DEPRECATED - Please use sourceMapTarget
    --source-map-target [string]         set `file` on returned source map
    --source-file-name [string]          set `sources[0]` on returned source map
    --source-root [filename]             the root from which all sources are relative
    --module-root [filename]             optional prefix for the AMD module formatter that will be prepend to the filename on module definitions
    --babelrc [list]                     Specify a custom list of babelrc files to use
    --source-type [string]
    -x, --extensions [extensions]        List of extensions to compile when a directory has been input [.es6,.js,.es,.jsx]
    -w, --watch                          Recompile files on changes
    -o, --out-file [out]                 Compile all input files into a single file
    -d, --out-dir [out]                  Compile an input directory of modules into an output directory
    -D, --copy-files                     When compiling a directory copy over non-compilable files
    -q, --quiet                          Don't log anything
    -V, --version                        output the version number

  Transformers:

    - [asyncToGenerator]
    - [bluebirdCoroutines]
    - es3.memberExpressionLiterals
    - es3.propertyLiterals
    - es5.properties.mutators
    - es6.arrowFunctions
    - es6.blockScoping
    - es6.classes
    - es6.constants
    - es6.destructuring
    - es6.forOf
    - es6.literals
    - es6.modules
    - es6.objectSuper
    - es6.parameters
    - es6.properties.computed
    - es6.properties.shorthand
    - es6.regex.sticky
    - es6.regex.unicode
    - [es6.spec.arrowFunctions]
    - [es6.spec.blockScoping]
    - [es6.spec.modules]
    - [es6.spec.symbols]
    - [es6.spec.templateLiterals]
    - es6.spread
    - es6.tailCall
    - es6.templateLiterals
    - [es7.asyncFunctions]
    - [es7.classProperties]
    - [es7.comprehensions]
    - [es7.decorators]
    - [es7.doExpressions]
    - [es7.exponentiationOperator]
    - [es7.exportExtensions]
    - [es7.functionBind]
    - [es7.objectRestSpread]
    - [es7.trailingFunctionCommas]
    - [eval]
    - flow
    - [jscript]
    - [minification.constantFolding]
    - [minification.deadCodeElimination]
    - [minification.memberExpressionLiterals]
    - [minification.propertyLiterals]
    - [minification.removeConsole]
    - [minification.removeDebugger]
    - [optimisation.flow.forOf]
    - [optimisation.modules.system]
    - [optimisation.react.constantElements]
    - [optimisation.react.inlineElements]
    - react
    - react.displayName
    - [reactCompat]
    - regenerator
    - [runtime]
    - spec.blockScopedFunctions
    - spec.functionName
    - [spec.protoToAssign]
    - [spec.undefinedToVoid]
    - strict
    - [utility.inlineEnvironmentVariables]
    - validation.react
    - [validation.undeclaredVariableCheck]

  Module formatters:

    - amd
    - amdStrict
    - common
    - commonStrict
    - ignore
    - system
    - umd
    - umdStrict

$ babel source/ --watch --out-dir build/

{%endhighlight%}

我们可以看到，Babel 支持的功能多得多，从 JSX 转译为 JavaScript 只是其中的一部分功能而已，其实这也意味着通过 Babel Transpiler，我们可以在 React.js 应用中使用 ES2015 中引入的新特性。另外，Babel 也支持和 `JSXTransformer.js` 一样的在客户端转译的功能，我们需要 Babel 安装文件中一个叫做 `browser.js` 的源文件。和 `JSXTransfomer.js` 一样，我们需要把这个文件拷贝到项目目录中：

{%highlight bash%}
cp /usr/local/lib/node_modules/babel/node_modules/babel-core/browser.js ~/source/react-startkit/babel/browser.js
{%endhighlight%}

{%highlight html lineno%}
<script src="babel/browser.js"></script>
{%endhighlight%}

这样我们就不需要在试用 React.js 之前就编写繁杂的构建脚本了。


## 小结

本文通过一个 Hello world 简单介绍了如何用纯粹的 JavaScript 和 JSX 来利用 React.js 来编写 Web 应用，大致提及React.js 中组件的概念，并介绍了 JSX。另外介绍了几种把 JSX 转译为 JavaScript 代码的办法。如果想了解关于 React.js 的技术细节，可以参考 [React.js 主页](https://facebook.github.io/react/index.html)。

另外本文所展示的示例仅仅是一个简单的静态应用，实际项目显然不会如此。我们需要存储和处理不同的状态，不管它是应用内部的状态，还是对外部公开的状态。后续我会继续写一些文章来介绍如何在 React.js 应用中处理 `state` 和 `prop`，它们分别对应私有状态和公有状态。另外后续也会介绍 React 的架构 [Flux](https://facebook.github.io/react/docs/flux-overview.html)。

## 资源

[React.js 主页](https://facebook.github.io/react/index.html)

[Babel 主页](https://babeljs.io/)

[React: Up & Running](http://shop.oreilly.com/product/0636920042266.do)

[谈谈React.js的核心入门知识](http://wwsun.me/posts/react-getting-started.html)