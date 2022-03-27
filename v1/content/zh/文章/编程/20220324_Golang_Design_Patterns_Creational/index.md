---
title: "Golang|设计模式-创造"
date: 2022-03-12T22:41:00+08:00
draft: false
分类: ["编程"]
标签: ["Golang", "设计模式"]
# weight: 1
cover:
    image: "https://wallpaperaccess.com/full/4482741.png"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: "\"When is the last time you look into the sky?\""
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "用Golang来实现创造设计模式"
author: "AidenCLX"
translationKey: "GolangDesignPatternsCreational"
---
已经是三月了，我必须开始寻找工作。算到现在我已经用golang一年左右了，而且我认为现在是时候实现一下那些经典的设计模式作为练习。之前曾有个[github仓库](https://github.com/tmrts/go-patterns)试着做了这些工作，但最迟的PR也已经是在五年前的了，这意味着这个仓库已经年久失修。因此我决定自己把这些事情再做一遍。但一就要感谢一下这个[github仓库](https://github.com/tmrts/go-patterns)，我能够借由此来找到许多有用的链接与资料，这节省了我许多再去寻找相关资料的时间。

这篇文章主要是关于创造设计模式。其他的设计模式将会留在其他的文章讲述。你可以在[这里](https://github.com/AidenKitamura/go-patterns)找到我的代码。

> __注意：__ 由于中文计算机术语之间的惯例存在较大差异，本文对于术语将会继续使用英语。

---

## 关于创造设计模式

正如他的名字一样，这些设计模式是用来创造实例的。举个例子，Singleton是用来限制实例数量的。

---

## Singleton

Singleton是面向对象语言中最为广泛使用的设计模式之一。他对类添加了限制，即对于同一个程序而言，某个类只能够拥有一个实例。

为了理解为什么我们需要Singleton，我们来看个例子。假设我们现在有一个ui类以及若干其他类。为了改变ui的视觉效果，我们应当只改变一个，且仅一个ui实例当中的参数。创造多个ui实例来在同一个输出设备上输出内容并没有任何的意义。因而我们可以使用一种限制实例数量的方法来控制这个类，而这种方法被叫做Singleton。

按照惯例，实现Singleton的类的构造器将是private的，并且一个```getInstance()```的方法将会是public的。

> __注意:__ Go并没有一个完全对应类的概念。因此我们可以使用一个*unexported struct*，和一个定义了所有方法的、公开的接口一起来作为类的定义。

首先，让我们来创造一个样例类和他的方法。它可以是一个非常简单的、仅有getter和setter的类，但你能够明白其中的概念。

```go
type singleton struct {
	x int
	y int
}

func (p *singleton) GetX() int {
	return p.x
}

func (p *singleton) GetY() int {
	return p.y
}

func (p *singleton) SetX(newX int) {
	p.x = newX
}

func (p *singleton) SetY(newY int) {
	p.y = newY
}
```

只将这些方法暴露给用户是可行的。用户依旧可以对变量进行操作。但如此，用户将不能够显式地指定变量的类型。那如果用户需要显式地使用变量的类型呢？比如，将实例传入作为函数的参数？我们可以创造一个公开的接口来帮助我们完成这份工作：

```go
type Singleton interface {
	GetX() int
	GetY() int
	SetX(newX int)
	SetY(newY int)
}
```

在这些之后，我们需要一个不对外公开的变量来保存这个实例。为了只实例化一次，我们可以使用```sync.Once```来帮助我们。因此我们如此定义变量：

```go
var _singleton_once sync.Once
var singletonInstance Singleton
```

好了！我们几乎完成了。还剩下什么？很简单，我们需要那个惯例的```getInstance()```方法。那就来吧！

```go
func GetInstance() Singleton {
	if singletonInstance == nil {
		_singleton_once.Do(
			func() {
				singletonInstance = &singleton{0, 0}
			})
	}
	return singletonInstance
}
```

就是这样了！当然我们也可以使用其他的方式，比如使用```sync.Mutex```。我不会说```sync.Mutex```是个不好的办法，但他非常昂贵，而且如果你在判断nil之后才上锁的话，可能会出现竞争条件。所以我个人偏好于使用```sync.Once```，我认为这是一个更好的办法。

---

## Factory

Factory将构造实例的过程隐藏起来，代替用户来构造实例。这个设计模式在有许多类似类的时候十分有用。

为了将工厂生产的产品返回，我们创造一个包含了所有所需方法的```Product```接口，以及我们的样例类：

```go
type Product interface {
	Introduce()
}

type display struct {
	description string
}

type phoneScreen struct {
	description string
}

func (d display) Introduce() {
	fmt.Println(d.description, "Display!")
}

func (p phoneScreen) Introduce() {
	fmt.Println(p.description, "PhoneScreen!")
}
```

太好了！还剩下什么？需要Factory类和对应的构造方法。但在Golang中，有点不一样。

> __注意:__ Go没有传统的class关键词，所以替代类当中的static方法（其实就是函数），我们可以直接使用一个函数。

```go
func Factory(t string) (p Product, err error) {
	switch t {
	case "display":
		p = display{"I am a 27 inch"}
	case "phoneScreen":
		p = phoneScreen{"I am a 5 inch"}
	default:
		err = fmt.Errorf("Invalid Product Type: %s", t)
	}
	return p, err
}
```

好了！这很简单。

---

## Builder

这个设计模式适用于创建复杂物体。让我们来创造一个复杂物体：

- 相机
	- 镜头
		- 变焦镜头
			- 焦段
			- 光圈
		- 定焦镜头
			- 焦段
			- 光圈
	- 机身
		- 快门
			- 电子快门
			- 焦平面快门
		- 传感器
			- CMOS
			- CCD
		- 材料
			- 塑料
			- 金属

由此写出类的定义：

```go
type camera struct {
	lens lens
	body body
}

type lens struct {
	lensType    string
	focalLength string
	aperture    float32
}

type body struct {
	shutter  string
	sensor   string
	material string
}

func (c camera) Briefing() {
	fmt.Println(c.Assemble())
}

func (c camera) Assemble() string {
	return "Assmbled Camera:\n" + fmt.Sprintf("Lens:\n\t%s\n\t%s\n\t%.1f\n", c.lens.lensType, c.lens.focalLength, c.lens.aperture) + fmt.Sprintf("Body:\n\t%s\n\t%s\n\t%s\n", c.body.shutter, c.body.sensor, c.body.material)
}

type Camera interface {
	Briefing()
	Assemble() string
}
```

接着我们需要builder类来帮助创建这些物体。我们给了两个方法，```AddLens```和```AddBody```，以此用户可以分别添加镜头和机身。如下所示：

```go
type builder struct {
	cam camera
}

type Builder interface {
	AddLens(lensType, focalLength string, aperture float32)
	AddBody(shutter, sensor, material string)
	GetProduct() Camera
}

func (b *builder) AddLens(lensType, focalLength string, aperture float32) {
	b.cam.lens = lens{lensType: lensType, focalLength: focalLength, aperture: aperture}
}

func (b *builder) AddBody(shutter, sensor, material string) {
	b.cam.body = body{shutter: shutter, sensor: sensor, material: material}
}

func (b *builder) GetProduct() Camera {
	return b.cam
}

func GetBuilder() Builder {
	cam := camera{}
	return &builder{cam: cam}
}
```

完成！

---

## Abstract Factory

与factory模式非常的类似，abstract factory模式也是用于创建实例的。区别在于，abstract factory本身就是一层抽象，而factory模式中factory是concrete的。因此在这里我们用一个接口来表示abstract factory：

```Go
type AbstractFactory interface {
	ProducePhone() Phone
}
```

然后我们加入需要的工厂以及他们的产品和制造办法，当然是private的。

```Go
type Phone struct {
	brand string
}

type samFactory struct {
}

type applFactory struct {
}

func (s samFactory) ProducePhone() Phone {
	return Phone{"Sam"}
}

func (a applFactory) ProducePhone() Phone {
	return Phone{"Appl"}
}
```

因为我们抽象了工厂，我们需要一个额外的函数来帮助我们获取这些工厂：

```Go
func GetAbstractFactory(brand string) (AbstractFactory, error) {
	switch brand {
	case "Sam":
		return samFactory{}, nil
	case "Appl":
		return applFactory{}, nil
	}
	return nil, fmt.Errorf("invalid brand name %s", brand)
}
```

好了！

---

## Prototype

Prorotype是用于帮助复制实例的。当一个实例非常复杂，亦或有一些private的变量，提供一个```Clone()```方法是非常有用的，由此新的实例拷贝就能够被方便的获取。

我们用链表来举例子。当复制链表时，我们不能够直接使用第一个元素的指针拷贝作为返回值，这样会带来一个问题，即后面的元素依旧指向原来的元素。我们对他们做修改的话，会在新的复制链表中体现出来。让我们先来定义一个链表：

```go
type linkedList struct {
	val  int
	next *linkedList
}

func (l *linkedList) GetVal() int {
	return l.val
}

func (l *linkedList) Next() LinkedList {
	return l.next
}

func (l *linkedList) Clone() LinkedList {
	startNode := &linkedList{}
	cur := startNode
	for ptr := l; ptr != nil; ptr = ptr.next {
		cur.val = ptr.val
		if ptr.next != nil {
			cur.next = &linkedList{}
			cur = cur.next
		} else {
			cur.next = nil
		}
	}
	return startNode
}
```

就像我们过去所做的，这```linkedList```是private的。因此我们需要一个接口来帮助我们：

```go
type LinkedList interface {
	GetVal() int
	Next() LinkedList
	Clone() LinkedList
}
```

> __注意：__ Golang对于接口的实现判定有一些小问题。请看上面的代码，```Next()```方法和```Clone()```方法均有```LinkedList```的返回类型，这一点都不优雅。但在Golang中接口的返回类型必须和函数的返回类型保持一致才被认为是实现了接口。在创建递归的结构时，你不能够用结构的指针本身作为返回类型。

---

## Object Pool

Object pool是一个先初始化一个池，然后当用户需要使用实例时直接从中获取的设计模式。这个模式在初始化实例非常昂贵，或实例几乎不会变化时，非常有用。

我们先创建一个样例object：

```go
type object struct {
	name string
}

func (o object) GetName() string {
	return o.name
}

type Object interface {
	GetName() string
}
```

在我的例子中，我使用了Singleton模式来确保我们不会多此初始化池。然而这并不是必须的，根据需要我们也可以初始化多个池来使用：

```go
var once sync.Once

var P *pool

type pool struct {
	*sync.Mutex
	idle   []object
	active []object
}

func (p *pool) Acquire(name string) (Object, error) {
	p.Lock()
	defer p.Unlock()
	for i := 0; i < len(p.idle); i++ {
		if p.idle[i].name == name {
			res := p.idle[i]
			p.idle[i] = p.idle[len(p.idle)-1]
			p.idle = p.idle[:len(p.idle)-1]
			p.active = append(p.active, res)
			return res, nil
		}
	}
	return nil, fmt.Errorf("cannot find object with name %s", name)
}

func (p *pool) Free(name string) error {
	p.Lock()
	defer p.Unlock()
	for i := 0; i < len(p.active); i++ {
		if p.active[i].name == name {
			res := p.active[i]
			p.active[i] = p.active[len(p.active)-1]
			p.active = p.active[:len(p.active)-1]
			p.idle = append(p.idle, res)
			return nil
		}
	}
	return fmt.Errorf("cannot free object with name %s", name)
}

type Pool interface {
	Acquire(name string) (Object, error)
	Free(name string) error
}

func InitPool(cap int) Pool {
	once.Do(func() {
		P = &pool{}
		P.Mutex = new(sync.Mutex)
		P.active = make([]object, 0)
		P.idle = make([]object, 0)
		for i := 0; i < cap; i++ {
			P.idle = append(P.idle, object{fmt.Sprintf("%d", i)})
		}
	})
	return P
}
```

感谢您阅读这篇文章！其他的设计模式将在未来的其他文章中讨论。