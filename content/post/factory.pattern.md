+++
author = "Izzy Mukhitdinov"
title = "Abstract Factory Design Pattern"
date = "2022-11-18"
description = "Brief look into the implementation of abstract factory design pattern in Go"
tags = [
    "design patterns",
    "design",
    "pattern",
    "abstract factory pattern",
    "factory pattern",
]
categories = [
    "design patterns",
]
series = []
aliases = []
+++

---

> "The noblest pleasure is the joy of understanding."<br>
> --<cite>Leonardo da Vinci</cite>

Today, I will be writing about `Abstract Factory` design pattern. To understand it better, I will be using Go programming language, instead of talking all theory.

Simply put, `Abstract Factory` pattern is a factory of factories. This begs a question of "What is a factory ?". `Factory` pattern is one of the most common patterns in Software Development and it is used to create objects of classes without exposing inner logic of classes to create objects. It provides an interface which can be used to create and access objects. That is simple, right ? Even though Go is not object-oriented programming language and does not have classes, we can use Go's structs, because they can have an object-like behavior.

Now let's try to see what `Factory` pattern looks like when applied to Go:

---
`main.go`

```go
package main

import (
	"fmt"
	"log"
)

type product struct {
	name  string
	price int
}

func (p *product) sell() {
	fmt.Printf("Sold: %s at price $%d\n", p.name, p.price)
}

func (p *product) buy() {
	fmt.Printf("Bought: %s at price $%d\n", p.name, p.price)
}

type cap struct {
	product
}

type tShirt struct {
	product
}

// 'i' indicates that the iProduct is an interface
type iProduct interface {
	sell()
	buy()
}

func factory(productName string) (iProduct, error) {
	if productName == "cap" {
		return &cap{
			product: product{
				name:  "Cap",
				price: 50,
			},
		}, nil
	}

	if productName == "t-shirt" {
		return &tShirt{
			product: product{
				name:  "T-Shirt",
				price: 72,
			},
		}, nil
	}

	return nil, fmt.Errorf("unknown product name %s", productName)
}

func main() {
	myCap, err := factory("cap")
	if err != nil {
		log.Fatal(err)
	}

	shirt, err := factory("t-shirt")
	if err != nil {
		log.Fatal(err)
	}

	myCap.sell()
	shirt.buy()
}

```

```bash
$ go run main.go

Sold: Cap at price $50
Bought: T-Shirt at price $72

```

---

The type `product` here represents the commonality of all possible products [`cap` and `t-shirt` in our example].

I deliberately avoided implementing the `iProduct` interface on types `cap` and `t-shirt`, since it would mean we will have to repeat ourselves again and again
every time, we introduce a new type of product.
Embedding the type `product` is a better solution, since it will make sure that our code satisfies the `Liskov Substitution`
principle.

[I will write about `SOLID` design principles later in other articles] 

If you look closely, you will realize that I also did not use any concrete type as the first return value of `factory` function.
The reasoning behind that is that it would make the code not flexible: What if there is a kind of product with a feature other kinds of products do not have ?
Hence the use of an interface.

Now the basic `Factory` pattern is done. `Abstract Factory` pattern is also a `Factory` pattern; It is just a factory of factories.

---
`factory.go`

```go
package main

type factory interface {
	makeCar() iCar
}

// 'i' indicates iCar is an interface
type iCar interface {
	drive()
}

func getFactory(brand string) factory {
	if brand == "bmw" {
		return bmw{}
	}

	if brand == "toyota" {
		return toyota{}
	}

	return nil
}

```

---
`car.go`

```go
package main

import "fmt"

type car struct {
	brand string
	color string
}

func (c car) drive() {
	fmt.Printf("%s %s is roaring...\n", c.color, c.brand)
}

```

---
`bmw.go`

```go
package main

type bmw struct {
}

type bmwCar struct {
	car
}

func (factory bmw) makeCar() iCar {
	return bmwCar{
		car: car{
			brand: "BMW",
			color: "Off-White",
		},
	}
}

```

---
`toyota.go`

```go
package main

type toyota struct {
}

type toyotaCar struct {
	car
}

func (factory toyota) makeCar() iCar {
	return toyotaCar{
		car: car{
			brand: "TOYOTA",
			color: "Red",
		},
	}
}

```

---
`main.go`

```go
package main

import "log"

func main() {
	bmwFactory := getFactory("bmw")
	if bmwFactory == nil {
		log.Fatal("Unknown brand name...")
	}

	toyotaFactory := getFactory("toyota")
	if toyotaFactory == nil {
		log.Fatal("Unknown brand name...")
	}

	bmwCar := bmwFactory.makeCar()
	toyotaCar := toyotaFactory.makeCar()

	bmwCar.drive()
	toyotaCar.drive()
}

```

```bash
$ go run *.go

Off-White BMW is roaring...
Red TOYOTA is roaring...

```

<br>

# Until the next time <span class="emojify">&#x1F60A;</span>