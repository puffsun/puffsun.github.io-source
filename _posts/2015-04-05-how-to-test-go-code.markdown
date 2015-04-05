---
layout: post
title: "如何测试 Go 代码 - 单元测试"
date: 2015-04-05T15:15:51+08:00
comments: true
categories: ["Go"]
keywords: Testing Test "Unit Test" "Integration Test" Go Golang 测试 单元测试 集成测试 httptest http goconvey Ginkgo GΩmega Gomega
description: Test Go code with build-in testing library and third-party test library
---

### 软件测试的重要性
关于软件测试是否重要，我想大家都有共识。ThoughtWorks 的大牛Michael C. Feathers在他的成名作 [Working Effectively with Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052) 的序言写道：

>对于我来说，遗留代码 (Legacy Code) 简单来讲就是没有测试覆盖的代码。不管它写得多好，也不管它面向对象封装得有多精美，没有测试覆盖的代码都是烂代码。有了测试覆盖，我们可以在可被验证的情况下快速的改变我们代码的行为，而如果没有测试覆盖，我们无从得知我们的代码是在变好还是在变坏。

随便列几条好处：

* 更容易也更有信心来对现有代码进行重构，没有单元测试覆盖的重构就像是在沼泽地行军，你根本无从知道下一步将会遇到什么；
* 在代码层面建立项目文档，可以通过阅读测试来了解整个项目是如何工作的；
* 通过提高测试覆盖，发现 bug；
* 通过单元测试来给代码解耦，通常写单元测试的时候，你需要建立一些外部依赖。如果在某个时候你发觉无法正确的建立单元测试的依赖，那么这时候你要反思项目的架构是不是出了问题；
* 通过视觉回馈，在编码的时候给你更多的信心；
* 。。。

那为什么还有很多程序员从来不写测试呢？其实在我看来原因有两条：

1. *懒*
2. 还是*懒*

### 写这篇文章的目的
在这篇文章里我来介绍一下 Go 语言对单元测试的支持，并会顺便介绍两个我自己用过的测试类库，分别是[Goconvey](https://github.com/smartystreets/goconvey)， [Ginkgo](https://github.com/onsi/ginkgo) 和 [GΩmega](https://github.com/onsi/gomega)。其他的 Go 第三方测试类库/Mock/Coverage 测试等等有很多，有兴趣可以参考[Go Testing Toolbox](http://nathany.com/go-testing-toolbox/)，这篇文章里面分门别类讲了不少，推荐阅读。

### 阅读本文的前提
在写测试之前，会写 Go 的代码一定是必要条件，如果你对 Go 语言还没有入门，那么你先移步 [Go by Example](https://gobyexample.com/) 或者 [A Tour of Go](http://tour.golang.org/)，这篇文章先放进你的 Evernote 或者 书签好了。

### 我们开始吧
假定我们手上有这么一段代码，我们将它保存为 `stringutil.go`（代码来自[golang/example](https://github.com/golang/example/blob/master/stringutil/reverse.go)）：
{%highlight go linenos%}
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
{%endhighlight%}

我们现在希望用单元测试覆盖这段代码，用Go 语言内置的 `testing` 库可以这么写单元测试：
{%highlight go linenos%}
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	for _, c := range []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	} {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
{%endhighlight%}
在代码的根目录下运行 `go test`，输出如下：
{%highlight bash%}
$ go test
PASS
ok      github.com/golang/example/stringutil    0.008s
{%endhighlight%}

这里有几点需要注意的：

1. Go 提供的运行测试代码的工具 `go test`，这个工具会自动寻找 filename_test.go 的测试代码并运行，这里我们应该给上面的测试代码命名为 `stringutil_test.go`。
2. Go 代码的测试用例必须以 `Test`起头才会被 `go test` 识别为一个合法的 Test Case。一个 Test Case 函数接受一个参数，类型为 `*testing.T`。至于 Test 后面的部分一般来说是我们想测试的函数，但这不是强制的。
3. `*testing.T`这个类型的主要目的是用来报告测试的错误，打印日志，在测试运行过程中日志会一直累积，并在测试代码运行结束后打印到控制台。
4. Go 还提供了 `*testing.B` 和 `*testing.M` 分别用于基准测试(Benchmark Testing)和做一些测试的设置和清理工作，具体可以参见[Go testing 文档](http://golang.org/pkg/testing/)。

大家可以看到，Go 语言虽然提供了丰富的测试支持，比如单元测试、基准测试、性能测试，以及把测试结果输出到控制台或者保存为其他格式，但是这种测试写起来还是有点别扭，不是我们常见的 xUnit 测试框架或者 BDD 测试风格。那接下来我要介绍一个风格是大家比较熟悉的测试框架，叫做[GoConvey](https://github.com/smartystreets/goconvey)，测试代码的风格有点类似于 Ruby 社区的 RSpec。

### GoConvey
我们来用 GoConvey 重写上面的单元测试：
{%highlight go linenos%}
package stringutil

import (
	. "github.com/smartystreets/goconvey/convey"
	"testing"
)

func TestSpec(t *testing.T) {

	// Only pass t into top-level Convey calls
	Convey("Given some ASCII and UTF8 strings", t, func() {
		strs := []struct {
			in, want string
		}{
			{"Hello, world", "dlrow ,olleH"},
			{"Hello, 世界", "界世 ,olleH"},
			{"", ""},
		}
		Convey("The value should be equal the reversed one", func() {
			for _, c := range strs {
				got := Reverse(c.in)
				So(got, ShouldEqual, c.want)
			}
		})
	})
}
{%endhighlight%}

这里我们需要注意的是：

1. 在 import 代码后面有个 `.`，这里的含义是把引入后的 Go `package` 中被导出(大写字母开头)的变量，结构体和函数直接放入执行 `import` 的 `package`，这样在引用的时候就不必要加 `package` 名作为前缀了。其他的方式还有 `import _ package_name` (仅仅运行 package 中的 `init` 方法来初始化), `import P package_name` (以 P 作为别名), 以及 `import package_name` (使用 package 来引用其中导出的元素)；
2. GoConvey 遵循 `go test` 一样的规则，这是因为它要和 `go test` 紧密集成；
3. GoConvey 还支持自动生成测试的一些样板代码(Boilerplate code)，也支持在浏览器中写测试，以及Chrome 浏览器通知等等高级特性，详细请查看[文档](https://github.com/smartystreets/goconvey)。

另外一个我也比较喜欢的测试框架是[Ginkgo](https://github.com/onsi/ginkgo)，[Gomega](https://github.com/onsi/gomega)是和 Ginkgo 结合得很紧密的匹配库(Matcher Library)，提供了很多人性化的匹配语法。在 Ginkgo 家族中还有另外的成员[Agouti](https://github.com/sclevine/agouti)，它主要用于集成测试(integration test)，我会在以后的博文中介绍。

### Ginkgo 和 Gomega
我们用 Ginkgo 和 Gomega 来改写上面的单元测试：
{%highlight go linenos%}
package stringutil_test

import (
	. "github.com/golang/example/stringutil"
	. "github.com/onsi/ginkgo"
	. "github.com/onsi/gomega"
)

var _ = Describe("StringutilTest", func() {
	var (
		strs []struct {
			in, want string
		}
	)

	BeforeEach(func() {
		strs = []struct {
			in, want string
		}{
			{"Hello, world", "dlrow ,olleH"},
			{"Hello, 世界", "界世 ,olleH"},
			{"", ""},
		}
	})

	Describe("With ASCII and UTF8 strings defined", func() {
		Context("Reverse the give strings", func() {
			It("should be reversed", func() {
				for _, c := range strs {
					got := Reverse(c.in)
					Expect(got).To(Equal(c.want))
				}
			})
		})
	})
})

{%endhighlight%}
这里需要注意的是：

1. 我们需要在项目根目录下运行 `ginkgo -r .` 来运行 Ginkgo 测试代码；
2. 我们在Ginkgo 测试里面使用的 package 名为 `stringutil_test`，这个命名有值得注意的地方。通常在 Go 代码中，我们会把代码和测试代码放在同样的 `package`下面，这样测试代码就可以直接访问需要测试的代码。但是有些情况下我们不能这么做，比如 在单元测试中我们需要打印测试结果，需要调用 `fmt` 的代码，而 `fmt` 包自身的代码也需要测试。Go 给出的解决方案是将测试代码的 `package` 命名为 `fmt_test`, 这样就同时导入了 `fmt` 和 `testing` 的代码，他们相互之间也避免了循环引用；
3. `Expect(got).To(Equal(c.want))` 风格的断言就来自于 Gomega；
4. Ginkgo 可以自动生成一些样板代码(Boilerplate code)，并可以监视文件系统 (ginkgo watch -r)，在文件变化以后自动运行测试，等等高级特性，详见[它的文档](http://onsi.github.io/ginkgo/)。

### 其他非常有价值工具

1. Go 还提供了检测并发代码竞争条件的工具，默认没有启用，可以通过 `go test -race` 来指定；
2. 另外，`go vet` 相当于代码的静态分析工具，如果集成到项目的 CI 中会对代码质量的保证大有帮助；
3. 其他工具，请见 Go [官方文档](https://golang.org/cmd/go/)，这里没提到不是因为它们不重要，而是有的我也不了解，:)


接下来本文的第二部分，我会介绍一下 Go 代码中如何写集成测试。

### 资源
[Go Testing Toolbox](http://nathany.com/go-testing-toolbox/)

[Ginkgo 主页](http://onsi.github.io/ginkgo/)

[Gomdga 主页](http://onsi.github.io/gomega/)

[Agouti 主页](http://agouti.org/)

[Go testing 官方文档](http://golang.org/pkg/testing/)

[GoConvey 主页](https://github.com/smartystreets/goconvey)

[Go Example 源代码](https://github.com/golang/example)

[Testing Techniques - Youtube 视频](https://www.youtube.com/watch?v=ndmB0bj7eyw)
