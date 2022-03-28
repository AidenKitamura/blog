---
title: "Golang | Design Patterns - Behavioral"
date: 2022-03-21T19:33:41+08:00
draft: false
categories: ["Programming"]
tags: ["Golang", "Design Patterns"]
# weight: 1
cover:
    image: "https://wallpaperaccess.com/full/5750684.jpg"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: ""
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "Implementing Behavioral Design Patterns using Golang"
author: "AidenCLX"
translationKey: "GolangDesignPatternsBehavioral"
---

Except for creational design patterns, there are also behavioral design patterns, which are mainly about methods. This is not a precise way to put it, but it tells something about behavioral design patterns: they are mainly about behaviors, and therefore quite procedural.

---

## Chain of Responsibility

Chain of responsibility is used to give a chain of processing processes, to process a request. For example, when compiling a program, we might need these steps:

1. Preprocesser to expand code
2. Compiler to generate assembly code
3. Assembler to generate object code
4. Linker to link with libraries and other files to generate the final executables

The above example might not be quite the meaning of chain of responsibility because a typical chain of responsibility design pattern will process the same object, while compiling a program will generate intermediate files to process. In my opinion, the chain of responsibility is like a subset of pipelines. If combined with go routines and channels, I believe this design pattern could be of great usage.

Let's use network response as an example:

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

Then we define several handlers:

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

Don't get confused by the name! These are not really parsers, but merely handlers. One very important part is that we use an ```interface``` to encapsulate all the handlers, and chain them together by using ```SetNext()``` function. That is the key idea.

---

## Command

Command design pattern, as the name says, is about sending and receiving commands. Take Bluetooth as an example, the receiver might be your headset while the invoker is your phone, the command being turning up volume, etc. The structure of this design pattern looks like:

- Invoker
    - Has Command Field
    - Has Command Sending Methods
- Command
    - Specifies What to Do
    - Embeds Receiver as Object
- Receiver
    - Implements Methods for Command to Call

Let's take aircraft as an example:

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

Then the command will be to take photos and fly, which can be written as:

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

And we will make the invokers, which are radio transmitters and phones. Here we use phones to take photos and radio transmitters to send flying signal:

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

So the chain here will be:
```Invokers sending Commands -> Commands received by Receivers -> Receiver carries out arrocdingly```

---

## Iterator

To be updated