---
title: "Golang | Design Patterns - Creational"
date: 2022-03-24T22:41:00+08:00
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