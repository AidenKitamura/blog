---
title: "Golang | Design Patterns - Creational"
date: 2022-03-12T22:41:00+08:00
draft: false
categories: ["Programming"]
tags: ["Golang", "Design Patterns"]
# weight: 1
cover:
    image: "https://wallpaperaccess.com/full/4482741.png"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: ""
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "Implementing Creational Design Patterns using Golang"
author: "AidenCLX"
translationKey: "GolangDesignPatternsCreational"
---
It is March now and I have to start looking for a job. I have been doing golang for about a year now, and I think it would be good to try to implement those nice design patterns using golang as a practice. There used to be one [github repo](https://github.com/tmrts/go-patterns) doing all these, but the latest pull request merged was 5 years ago, meaning that it was no longer maintained. Therefore, I think I will do these by myself. But thanks to the [github repo](https://github.com/tmrts/go-patterns), I can find a lot of links to useful resources, which saved a lot of work finding different articles covering everything.

This post is mainly about creational design patterns. Other design patterns will be covered in other posts. You can find my code [here](https://github.com/AidenKitamura/go-patterns).

---

## About Creational Design Patterns

As the name says, these design patterns are mainly about creating objects. For example, singleton would limit the number of instances of a certain class to one, etc.

---

## Singleton

Singleton is one of the most widely used design patterns in OOP. It adds constraints to a certain class, namely the class could only have one instances as long as there is one program running.

In order to understand why we need singleton, let take an example. Say now we have an application consisting of a ui class and several other classes. In order to change the ui components, we should only call to one, and the only one ui instance. It makes nonsense creating several ui instances, operating on the output devices (displays or anything) concurrently, with different parameters. Therefore we have to come up with a way to restrain programmers from instantiating more than one instances of this class, and hola! Here comes the singleton design pattern.

By convention, to implement the singleton pattern, the constructor of the class will be private and a ```getInstance()``` method will be made public.

> __NOTE:__ Go doesn't have a corresponding concept for class. Therefore we have to use an _unexported structure_ as the class definition, as well as an interface exposing all methods we would like the user to use.

So first of all, let's create a mock class with its corresponding methods. It's a very simple class with only getters and setters, but you get the idea.

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

It is okay that we only expose these methods to the user. The user can still operate on the variable. It's just that the user cannot explictly assign type to that variable. But what if the user need to explicitly access the type to - maybe pass the instance as a parameter? So we need to create an interface, which is exported, to get the job done:

```go
type Singleton interface {
	GetX() int
	GetY() int
	SetX(newX int)
	SetY(newY int)
}
```

And after these, we need a variable to store the instance. In order to initiate the instance only once, we can use ```sync.Once``` to help us. And therefore we declare these two variables:

```go
var _singleton_once sync.Once
var singletonInstance Singleton
```

That's it! We are almost done, so what's left? Simple, we need the conventional ```getInstance()``` method. So be it!

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

So that's how it works! There are also other ways to implement this, like using ```sync.Mutex```. I wouldn't say it is a bad way, but ```sync.Mutex``` is expensive, and there might be race conditions if you acquire the lock after the first nil checking condition. So I personally will stick to ```sync.Once```, which in my opinion is a better approach.

---

## Factory

Factory hides the constructing details from the user, helping the user to create instances. This design pattern is especially useful when there are similar classes.

In order to return the products, let's create a ```Product``` interface which includes every methods we need, as well as some sample product classes:

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

Great! Now what's left? The Factory class with a corresponding method! But in Golang, things are a bit different.

> __NOTE:__ Go doesn't have a traditional "class" keyword, so instead of using a class with static methods (which are literally functions), it is easier just put an exported function in go as the factory.


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

Done! This one is simple.

---

## Builder

This design pattern is used to construct complicated objects. Let's think of one complicated objects first:

- Camera
	- Lens
		- Zoom Lens
			- Focal Length Range
			- Aperture Range
		- Prime Lens
			- Focal Length
			- Aperture Range
	- Body
		- Shutter
			- Electronic Shutter
			- Focal Plane Shutter
		- Sensor
			- CMOS
			- CCD
		- Material
			- Plastic
			- Metal

Here are the class definitions:

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

Then we need a builder class to help construct these objects. There are two methods, one ```AddLens```, one ```AddBody```, so that the user can initiate these two separately. As shown below:

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

Done!

---

## Abstract Factory

Being very similar to the factory design pattern, abstract factory also provides an abstration to create instances. The difference is that the abstract factory itself is an abstraction, whereas the factory is concrete. So here we use an interface to represent the abstract factory:

```Go
type AbstractFactory interface {
	ProducePhone() Phone
}
```

Then we need to add the wanted concrete factories, private ones, of course, with their products and manufacturing methods.

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

Since we encapsulated the factories, we need an extra function to help us get these factories:

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

Done!

---

## Prototype

Prorotype is used to help create cloned objects. When an object is complicated or has some private fields, it would be good to provide a clone method with it so that a new object with the same fields could be created.

Let's use linked list as an example. When cloning a linked list, we could not just use the pointer to the first element, or only "clone" the first element, which will cause serious problem because of in-place operations. Let's define a linked list first:

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

As we did before, the ```linkedList``` is private. In this case, we need an interface to operate on this:

```go
type LinkedList interface {
	GetVal() int
	Next() LinkedList
	Clone() LinkedList
}
```

> __NOTE:__ There is a problem with Golang's interface implementation judgement. Take a look at the code above, the ```Next()``` method and ```Clone()``` method have a return type of ```LinkedList```, which is not beautiful because in Golang, the return type of a method from an interface must match with the method's return type from a concrete type. You cannot use the type of the struct as the return type.

---

## Object Pool

Object pool is a pattern that one initialize a pool of various objects first, then whenever the user needs one, he/she acquire an object from the pool and use it. This pattern is especially useful when initializing an object is expensive or object isn't subject to changes, but only usages.

We create the sample objects first:

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

In my example I adopted singleton to make sure we don't re-initialize the pool. However it is possible when we need multiple pools, we initialize them respectively:

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

Thank you for reading this post. More design patterns will be covered in other posts in the future!