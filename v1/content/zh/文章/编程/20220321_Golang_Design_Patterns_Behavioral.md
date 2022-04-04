---
title: "Golang | 设计模式 - 行为"
date: 2022-03-21T19:33:41+08:00
draft: false
分类: ["编程"]
标签: ["Golang", "设计模式"]
# weight: 1
cover:
    image: "https://wallpaperaccess.com/full/5750684.jpg"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: ""
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "用Golang实现行为设计模式"
author: "AidenCLX"
translationKey: "GolangDesignPatternsBehavioral"
---

除了创造设计模式之外，设计模式中还包含行为设计模式，而这些设计模式主要是关于方法使用的。这个表述并不准确，但它确实告诉了你关于行为设计模式的一些特点：他们是关于行为、流程设计的。

> __注意：__ 由于中文计算机术语之间的惯例存在较大差异，本文对于术语将会继续使用英语。

---

## Chain of Responsibility

Chain of responsibility被用于链式地处理一个对象，比如一个请求。举个例子，当我们编译一个程序时，可能会有如下几步：

1. 预处理
2. 生成汇编代码
3. 生成目标码
4. 链接库和其他文件来生成可执行文件

上述的例子可能并非是chain of responsibilities的最好体现，一个典型的chain of responsibilities的模式常常是处理同一个目标，而编译程序往往是生成中间的文件，再对中间的文件进行相应的处理。在我看来，chain of responsibilities像是管线的一个子集；如果能够用好goroutine和channels，我认为这个设计模式将会由很大的用处。

我们用网络回应来举个例子：

```go
type networkResponse struct {
	status int
	header string
	body string
}

func (n networkResponse) GetStatus() int {
	return n.status
}

func (n networkResponse) GetHeader() string {
	return n.header
}

func (n networkResponse) GetBody() string {
	return n.body
}

type NetworkResponse interface {
	GetStatus() int
	GetHeader() string
	GetBody() string
}
```

然后定义一些handlers：

```go
type Parser interface {
	Parse(n NetworkResponse)
	SetNext(p Parser)
}

type statusParser struct {
	next Parser
}

func (s *statusParser) Parse(n NetworkResponse) {
	switch n.GetStatus() {
	case 404:
		fmt.Println("Cannot find page, abort.")
	default:
		fmt.Printf("Other status: %d\n", n.GetStatus())
		if s.next != nil {
			s.next.Parse(n)
		}
	}
}

func (s *statusParser) SetNext(p Parser) {
	s.next = p
}

type headerParser struct {
	next Parser
}

func (h *headerParser) Parse(n NetworkResponse) {
	switch n.GetHeader() {
	case "Text", "CSS", "JS":
		fmt.Printf("Got Header: %s\n", n.GetHeader())
		if h.next != nil {
			h.next.Parse(n)
		}
	default:
		fmt.Printf("Unknown header, abort.\n")
	}
}

func (h *headerParser) SetNext(p Parser) {
	h.next = p
}

type bodyParser struct {
	next Parser
}

func (b *bodyParser) Parse(n NetworkResponse) {
	fmt.Println(n.GetBody())
}

func (b *bodyParser) SetNext(p Parser) {
	b.next = p
}
```

不要被这些名字弄混淆了！他们其实并不是parser，只是一些handler而已。重要的是，我们利用一个```interface```来抽象化所有的handlers，并利用```SetNext()```把他们连接在一起。这才是最重要的部分。

---

## Command

Command设计模式，就像名字一样，是用来发送接受命令的设计模式。用蓝牙来举个例子，receiver可能是你的耳机，invoker是你的手机，而命令可能是增大音量，等等。这个设计模式的结构看起来就像这样：

- Invoker
    - Has Command Field
    - Has Command Sending Methods
- Command
    - Specifies What to Do
    - Embeds Receiver as Object
- Receiver
    - Implements Methods for Command to Call

我们用无人机来做样例：

```go
type aircraft struct {
}

func (a aircraft) Fly() {
	fmt.Println("Flying!")
}

func (a aircraft) TakePhoto() {
	fmt.Println("Taking Photos!")
}

type Receiver interface {
	Fly()
	TakePhoto()
}
```

命令，我们使用飞行和拍照作为样例：

```go
type fly struct {
	r Receiver
}

func (f fly) Execute() {
	f.r.Fly()
}

type takeShot struct {
	r Receiver
}

func (t takeShot) Execute() {
	t.r.TakePhoto()
}

type Command interface {
	Execute()
}
```

我们使用遥控器和手机分别操作飞行和拍照的命令：

```go
type radioTransmitter struct {
	command Command
}

func (r radioTransmitter) Send() {
	r.command.Execute()
}

type phone struct {
	command Command
}

func (p phone) Send() {
	p.command.Execute()
}

type Invoker interface {
	Send()
}
```

而整个流程看起来就像：
```Invokers sending Commands -> Commands received by Receivers -> Receiver carries out arrocdingly```

---

## Iterator

