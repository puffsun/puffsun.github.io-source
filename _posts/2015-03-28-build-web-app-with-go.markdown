---
layout: post
title: "用 Go 开发 Web 应用"
date: 2015-03-28T22:01:52+08:00
comments: true
categories: ["Go"]

keywords: "Go Golang web application software programming web应用 软件编程 tutorial 教程"
description: "Build Web Application with Golang"
---

### 为什么写这篇文章
学习 Go 语言到今天为止大约花了两周多的业余时间，在学习的过程中，先后参考了[Go By Example](https://gobyexample.com/)，用 Go 语言把[常规的算法和数据结构](https://github.com/puffsun/go_algorithms_data_structures)实现了一遍，并大致了解了如何用 Go 语言来开发 Web 应用。在学习如何用 Go 语言写 Web 应用的过程中，看到了[一篇 Go 官方的教程](https://golang.org/doc/articles/wiki/)，内容非常丰富，当时就有翻译这篇文章的念头。但是我又不想完整的翻译这篇文章，因为原文前半部分对程序讲解得非常详细，如果你知道 Go 的基础知识，很多内容可以跳过，再者我也不希望本文像原文那样长，所以这篇文章相当于我的学习笔记，或者对原文的总结了，如果你觉得本文步伐过大，导致有点蛋疼，那么就请移步[原文](https://golang.org/doc/articles/wiki/)。

### 内容

1. Go 开发环境参考：[安装](https://golang.org/doc/install)，[Go 项目文件结构](https://golang.org/doc/code.html)
2. Go 开发 Web 应用的工具类库：[net/http](https://golang.org/pkg/net/http/), [html/template](https://golang.org/pkg/html/template/), [regexp](https://golang.org/pkg/regexp/)
3. Closure

在阅读本文之前，你一定要有编程经验，要懂得基本的 Web 开发技术（HTML、HTTP），会使用命令行。

### 开始
{%highlight bash lineno%}
$ cd $GOPATH/src
$ mkdir gowiki
$ cd gowiki
{%endhighlight%}

创建名为 wiki.go 的文件，并键入如下内容：
{%highlight go lineno%}
package main

import (
	"fmt"
	"io/ioutil"
) 
{%endhighlight%}
在import 的两个 package 里面，[io/ioutil](https://golang.org/pkg/io/ioutil/)我们用于从磁盘读写文件，而 [fmt](https://golang.org/pkg/fmt/) 将会用于打印内容到标准输出，一般来说就是你键入命令的 Terminal.

接下来定义我们将使用的核心数据结构，以及用于将文件保存至磁盘和从磁盘加载的方法(method)，注意，这里不是函数(function)：
{%highlight go lineno%}
type Page struct {
    Title string
    Body  []byte
}

func (p *Page) save() error {
    filename := p.Title + ".txt"
    return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) *Page {
    filename := title + ".txt"
    body, _ := ioutil.ReadFile(filename)
    return &Page{Title: title, Body: body}
}

func main() {
    // 实例化一个 struct
    p1 := &Page{Title: "TestPage", Body: []byte("This is a sample Page.")}
    // 保存到磁盘
    p1.save()
    // 从磁盘加载
    p2, _ := loadPage("TestPage")
    // 打印到标准输出
    fmt.Println(string(p2.Body))
}
{%endhighlight%}

如果你对理解以上代码有难度，请移步原文，参考详细的解释。原文是英文的，如果你对英文理解有难度（作为一个程序员，不会英文其实是自废一半以上的武功），那么你可以跟我联系，我也很乐意帮你讲解。

这时候运行代码，就可以看到输出了：
{%highlight bash lineno%}
$ go run wiki.go
This is a sample page.
{%endhighlight%}

### 一个简单的 Go Web 应用
下面这段代码和上面我们完成的实例没有直接关系，只是用于演示如何用 Go 写 Web 应用。
{%highlight go lineno%}
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
{%endhighlight%}

可以看到我们在这里给 Web 应用的根路径绑定了一个 HTTP 请求处理器，这个处理器可以提取出请求路径参数，并打印一句包含这个参数的内容到命令行。至于http.ResponseWriter，http.Request，http.HandleFunc，和http.ListenAndServe, 都可以在[这里](https://golang.org/pkg/net/http/)找到对应的文档。

假定源文件名为 webapp.go, 这时如果你运行：`go run webapp.go`, 并访问如下链接，`http://localhost:8080/monkeys`，那么你将会看到如下输出：`Hi there, I love monkeys!`

### 使用 net/http 来开发 wiki
{%highlight go lineno%}
import (
	"fmt"
	"io/ioutil"
	"net/http"
)

type Page struct {
    Title string
    Body  []byte
}

func (p *Page) save() error {
    filename := p.Title + ".txt"
    return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) *Page {
    filename := title + ".txt"
    body, _ := ioutil.ReadFile(filename)
    return &Page{Title: title, Body: body}
}

// 用于处理以/view/起头的 HTTP 请求
func viewHandler(w http.ResponseWriter, r *http.Request) {
    //截取 HTTP 路径参数
    title := r.URL.Path[len("/view/"):]
    // 加载页面
    p, _ := loadPage(title)
    // 打印内容到 HTTP Response
    fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}

func main() {
    http.HandleFunc("/view/", viewHandler)
    http.ListenAndServe(":8080", nil)
}
{%endhighlight%}

在上面的代码里我们看到错误信息被用 `_` 人为的忽略了，在实际项目中我们绝对不可以这么做，这里只是为了简化代码，后面我们会处理这个返回的错误。

{%highlight bash lineno%}
$ go run wiki.go
{%endhighlight%}
访问http://localhost:8080/view/test，你就可以看到页面内容输出了。

### wiki编辑功能
一个不能编辑内容的 wiki 是没有任何实用价值的，接下来我们就要来开发 wiki 的编辑和保存功能。
{%highlight go lineno%}
import (
	"fmt"
	"io/ioutil"
	"net/http"
)

type Page struct {
    Title string
    Body  []byte
}

func (p *Page) save() error {
    filename := p.Title + ".txt"
    return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) *Page {
    filename := title + ".txt"
    body, _ := ioutil.ReadFile(filename)
    return &Page{Title: title, Body: body}
}

// 用于处理以/view/起头的 HTTP 请求
func viewHandler(w http.ResponseWriter, r *http.Request) {
    //截取 HTTP 路径参数
    title := r.URL.Path[len("/view/"):]
    // 加载页面
    p, _ := loadPage(title)
    // 打印内容到 HTTP Response
    fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}

func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    fmt.Fprintf(w, "<h1>Editing %s</h1>"+
        "<form action=\"/save/%s\" method=\"POST\">"+
        "<textarea name=\"body\">%s</textarea><br>"+
        "<input type=\"submit\" value=\"Save\">"+
        "</form>",
        p.Title, p.Title, p.Body)
}

func main() {
    http.HandleFunc("/view/", viewHandler)
    http.HandleFunc("/edit/", editHandler)
    http.ListenAndServe(":8080", nil)
}
{%endhighlight%}

有代码洁癖的读者到这里一定感觉到有浓浓的代码坏味道(code smell)扑鼻而来。我们在上面的代码中夹杂了很多 HTML 标签，看着就很恶心。这时候[html/template](https://golang.org/pkg/html/template/)就可以派上用场了。利用它，我们可以把 HTML 代码分离出去：
{%highlight go lineno%}
import (
	"html/template"
	"io/ioutil"
	"net/http"
)

type Page struct {
    Title string
    Body  []byte
}

func (p *Page) save() error {
    filename := p.Title + ".txt"
    return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) *Page {
    filename := title + ".txt"
    body, _ := ioutil.ReadFile(filename)
    return &Page{Title: title, Body: body}
}

// 用于处理以/view/起头的 HTTP 请求
func viewHandler(w http.ResponseWriter, r *http.Request) {
    //截取 HTTP 路径参数
    title := r.URL.Path[len("/view/"):]
    // 加载页面
    p, _ := loadPage(title)
    // 打印内容到 HTTP Response
    fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}

func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    t, _ := template.ParseFiles("edit.html")
    t.Execute(w, p)
}

func main() {
    http.HandleFunc("/view/", viewHandler)
    http.HandleFunc("/edit/", editHandler)
    http.ListenAndServe(":8080", nil)
}
{%endhighlight%}

创建名为 `edit.html` 的新文件：
{%highlight html lineno%}
<h1>Editing {{.Title}}</h1>

<form action="/save/{{.Title}}" method="POST">
<div><textarea name="body" rows="20" cols="80">{{printf "%s" .Body}}</textarea></div>
<div><input type="submit" value="Save"></div>
</form>
{%endhighlight%}

在上面的代码中，`template.ParseFiles` 读取 `edit.html` 的内容，并返回`*template.Template` 实例。`t.Execute` 运行模板，并将创建的 HTML 内容写入 `http.ResponseWriter`. 这里 `.Title` 和 `.Body` 分别指向 Page 对象里面的 Title 和 Body。

模板被`{% raw %}{{}}{% endraw %}`括起来。`http/template` 的另外一个好处是它可以自动转义生成的 HTML 代码，避免了 [XSS](http://en.wikipedia.org/wiki/Cross-site_scripting) 攻击。

接下来，我们也顺手把 `viewHandler` 利用 template 清理一下：

{%highlight go lineno%}
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    t, _ := template.ParseFiles("view.html")
    t.Execute(w, p)
}
{%endhighlight%}

view.html 文件内容：
{%highlight html lineno%}
<h1>{{.Title}}</h1>
<p>[<a href="/edit/{{.Title}}">edit</a>]</p>
<div>{{printf "%s" .Body}}</div>
{%endhighlight%}

这时候，嗅觉灵敏的读者又会发现，我们在 `viewHandler` 和 `editHandler` 里面对 template 的处理逻辑完全一样，本着 [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) 原则，我们这里要将重复的代码移到一个函数(function)里面，请注意，不是方法(method)！
{%highlight go lineno%}
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, _ := template.ParseFiles(tmpl + ".html")
    t.Execute(w, p)
}

func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}
{%endhighlight%}

### 对页面不存在错误的处理
这时候如果你访问 `/view/APageThatDoesntExist`，你会发现程序不会按照你预想的路径运行了，这是因为我们之前人为忽略了程序返回的错误码，这时候我们亡羊补牢还来得及，对这种情况，我们将会把页面重定向到新页面的编辑界面：
{%highlight go lineno%}
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}
{%endhighlight%}

### 保存编辑结果
{%highlight go lineno%}
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    p.save()
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
{%endhighlight%}
保存完毕，页面将会被重定向到查看页面。这里 FormValue 的返回值是 string, 我们利用 `[]byte(body)` 将其转换到 `[]byte`.

### 错误处理
在实际项目中，代码的错误需要被恰当的处理，这也就是我们将要做的事情：
{%highlight go lineno%}
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, err := template.ParseFiles(tmpl + ".html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    err = t.Execute(w, p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}

func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err := p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
{%endhighlight%}
注意这里 HTTP 错误码的运用。当然我们这里会把错误信息直接抛向用户，这也是很不友好的做法，实际项目中可以根据实际情况自定义友好的4xx和5xx 页面。

### 缓存模板
到目前为止，我们的代码有个重大的性能，每一次页面请求都会导致模板重新加载、解析，比较好的做法是在启动时把解析好的模板文件缓存起来。随后请求利用[ExecuteTemplate](https://golang.org/pkg/html/template/#Template.ExecuteTemplate)来渲染模板：

{%highlight go lineno%}
// 全局变量
var templates = template.Must(template.ParseFiles("edit.html", "view.html"))

func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    err := templates.ExecuteTemplate(w, tmpl+".html", p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
{%endhighlight%}

我们这里，模板的名字就是模板的文件名，因此在 `renderTemplate` 方法中我们需要拼接出完整的文件名。

### 正则表达式校验
到目前为止，我们的代码存在一个重大的安全问题，用户可以在浏览器中输入任何路径来读写服务器硬盘，这时候利用正则表达式校验用户输入就可以派上用场了。
{%highlight go lineno%}
import (
	"html/template"
	"io/ioutil"
	"net/http"
	"regexp"
	"errors"
)

// 用正则表达式匹配，并捕捉用户输入的路径参数，保证参数中只包含大小写字母和数字
var validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)$")

func getTitle(w http.ResponseWriter, r *http.Request) (string, error) {
    m := validPath.FindStringSubmatch(r.URL.Path)
    if m == nil {
        http.NotFound(w, r)
        return "", errors.New("Invalid Page Title")
    }
    // The title is the second subexpression.
    return m[2], nil
}

func viewHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err = p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
{%endhighlight%}

### 利用闭包来去除重复代码
仔细观察上面的代码，我们在每个处理器中的错误处理代码几乎一模一样。如果我们把错误处理代码集中起来，并在这个函数(function)中返回 `view`, `edit`, 和 `save` 处理器，那么就可以去除不少重复代码。Go 语言的 [function literals](https://golang.org/ref/spec#Function_literals) 在这里可以帮助我们。我们知道 Go 语言中函数(function)是一等公民，这有别于 Java/C# 等语言。

修改后的处理器函数签名如下：
{%highlight go lineno%}
func viewHandler(w http.ResponseWriter, r *http.Request, title string) {...}
func editHandler(w http.ResponseWriter, r *http.Request, title string) {...}
func saveHandler(w http.ResponseWriter, r *http.Request, title string) {...}

// 代码和 getTitle 基本相似
func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        m := validPath.FindStringSubmatch(r.URL.Path)
        if m == nil {
            http.NotFound(w, r)
            return
        }
        fn(w, r, m[2])
    }
}
{%endhighlight%}
从 `makeHandler` 中返回的闭包从HTTP 访问路径中抽取了 `title` 参数，随后通过正则表达式校验了它的内容。

最终完整的，可运行的代码如下：
{%highlight go lineno%}
package main

import (
	"errors"
	"flag"
	"html/template"
	"io/ioutil"
	"log"
	"net"
	"net/http"
	"regexp"
)

var (
	templates = template.Must(template.ParseFiles("edit.html", "view.html"))
	validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)$")
	addr      = flag.Bool("addr", false, "find open address and print to final-port.txt")
)

type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := ioutil.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}

func viewHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		http.Redirect(w, r, "/edit/"+title, http.StatusFound)
		return
	}
	renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		p = &Page{Title: title}
	}
	renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request, title string) {
	body := r.FormValue("body")
	p := &Page{Title: title, Body: []byte(body)}
	err := p.save()
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	http.Redirect(w, r, "/view/"+title, http.StatusFound)
}

func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
	err := templates.ExecuteTemplate(w, tmpl+".html", p)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
}

func getTitle(w http.ResponseWriter, r *http.Request) (string, error) {
	m := validPath.FindStringSubmatch(r.URL.Path)
	if m == nil {
		http.NotFound(w, r)
		return "", errors.New("Invalid Page Title")
	}
	return m[2], nil
}

func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		m := validPath.FindStringSubmatch(r.URL.Path)
		if m == nil {
			http.NotFound(w, r)
			return
		}
		fn(w, r, m[2])
	}
}

func main() {
	flag.Parse()
	http.HandleFunc("/view/", makeHandler(viewHandler))
	http.HandleFunc("/edit/", makeHandler(editHandler))
	http.HandleFunc("/save/", makeHandler(saveHandler))

	if *addr {
		l, err := net.Listen("tcp", "127.0.0.1:0")
		if err != nil {
			log.Fatal(err)
		}
		err = ioutil.WriteFile("final-port.txt", []byte(l.Addr().String()), 0644)

		if err != nil {
			log.Fatal(err)
		}
		s := &http.Server{}
		s.Serve(l)
		return
	}

	http.ListenAndServe(":8080", nil)
}
{%endhighlight%}

运行 `go run wiki.go`, 并访问 `http://localhost:8080`，你就会发现一个崭新的可以用来编辑、保存、查看的 wiki 了。

完整可运行的代码可见[我的 GitHub 代码库](https://github.com/puffsun/gowiki).

### 可以继续完善的地方

* 把模板保存到 `tmpl/` 下，把页面保存到 `views`下，把程序生成的数据保存到 `data/` 下；
* 为 wiki 增加根路径处理器，导航到 /view/FrontPage
* 为页面增加一些 CSS，使页面美观一些
* 简化站内页面之间的导航，就像其他的功能完备的 wiki 一样，支持 `[PageName]` 导航到 `<a href="/view/PageName">PageName</a>`, 可以尝试使用 `regexp.ReplaceAllFunc` 来实现这个功能。

### 资源

1. [Writing Web Applications with Go](https://golang.org/doc/articles/wiki/)
2. [Go Packages](https://golang.org/pkg/)
3. [Go by Example](https://gobyexample.com/)
4. [Go Algorithms and Data Structures - by George](https://github.com/puffsun/go_algorithms_data_structures)
5. [GoWiki完整源代码](https://github.com/puffsun/gowiki)

