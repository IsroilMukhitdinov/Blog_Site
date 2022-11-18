+++
author = "Izzy Mukhitdinov"
title = "Single Responsibility Principle"
date = "2022-11-18"
description = "Single responsibility principle in action"
tags = [
    "SOLID principles",
    "SOLID",
    "software design",
    "single responsibility",
]
categories = [
    "software design principles",
]
series = []
aliases = []
+++

---

> "The great art of learning is to understand but little at a time."<br>
> --<cite>John Locke</cite>

Today, I made an intention to cover all the SOLID design principles that was introduced by Robert C.Martin in his amazing piece of work, ` Agile Software Development, Principles, Patterns, and Practices`. But how do you eat an elephant ? One bite at a time. So, in this post, We will be exploring the first principle which is
`Single Responsibility` principle. Later in the series of software design principles, We will examine the rest of the principles.

One of the rules of Unix philosophy is to design software pieces that does only one thing and do it so well. The benefit is that you can gather all the small modules or units of code, and make them work together to accomplish a much greater task. That is a moduler design, which I love the most. If you want to learn more about it, go to [The Unix Operating System]("https://www.youtube.com/watch?v=tc4ROCJYbm0").

To see the `Single Responsibility` principle in action, let's write some code to find the square root of a given floating point number using [Newton's method]("https://en.wikipedia.org/wiki/Newton%27s_method") which basically uses successive guessing, going closer and closer to the right answer and stops when the guess is an enough approximation.

---
main.go

```go
package main

import (
	"errors"
	"fmt"
	"log"
)

func main() {
	num := 35.82

	result, err := squareRoot(num)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("the square root of %f is approximately %f\n", num, result)
}

func squareRoot(num float64) (float64, error) {
	if num < 0 {
		return 0, errors.New("cannot find the square root of a negative number")
	}

	guess := 1.0

	diff := guess*guess - num
	if diff < 0 {
		diff = -diff
	}

	for diff > 0.001 {
		guess = (guess + num/guess) / 2.0

		diff = guess*guess - num

		if diff < 0 {
			diff = -diff
		}
	}

	return guess, nil
}

```

```bash
$ go run main.go

the square root of 35.820000 is approximately 5.984981
```

If you double check the result, you will see that 5.984981 × 5.984981 = 35.81999757 which means program indeed produced a really close answer.

[Newton's method]("https://en.wikipedia.org/wiki/Newton%27s_method") is not just good for finding the square roots. It can also find the cubic root of a given number.
Let's see that

---
main.go

```go

package main

import  "fmt"

func main() {
	num := 35.82

	result := cubicRoot(num)
	fmt.Printf("the cubic root of %f is approximately %f\n", num, result)
}


func cubicRoot(num float64) float64 {
	guess := 1.0

	diff := guess*guess*guess - num
	if diff < 0 {
		diff = -diff
	}

	for diff > 0.001 {
		guess = (2.0*guess + num/(guess*guess)) / 3
		diff = guess*guess*guess - num

		if diff < 0 {
			diff = -diff
		}
	}

	return guess
}

```

```bash
$ go run main.go

the cubic root of 35.820000 is approximately 3.296415
```

Feel free to check the result. 3.296415 × 3.296415 × 3.296415 = 35.820005241

The problem with this style of code is that we are going to repeat ourselves every time we want a new function that finds the any kind of root of a given number. 
And when you find yourself doing the same thing over and over again, there is definitely something wrong with your approach. 
If you look closely, you will easily see many patterns lurking in our code. Every time we start with an initial guess and check if it is close enough answer. In case it is, we just return it. In case it is not, we update the guess based on a certain procedure. And the cycle keeps going until if an approximate answer is found.

The code is begging us to contain those patterns or logic in a small modules that can be used anywhere appropriate; We can combine all the necessary modules to accomplish the task of finding the square root, cubic root,...

The top most fuctions [squareRoot and cubicRoot] should not be RESPONSIBLE for doing everything

---
main.go

```go
package main

import "fmt"

func main() {
	number := 35.82

	result := squareRoot(number)
	fmt.Printf("Square root of %f is approximately %f\n", number, result)

	result = cubicRoot(number)
	fmt.Printf("Cubic root of %f is approximately %f\n", number, result)
}

func squareRoot(num float64) float64 {
	num = absoluteValue(num)
	guess := 1.0
	tolerance := 0.001

	for !isGoodEnough(guess, num, square, tolerance) {
		guess = improveGuess(guess, num, improveSquare)
	}

	return guess
}

func cubicRoot(num float64) float64 {
	guess := 1.0
	tolerance := 0.001

	for !isGoodEnough(guess, num, cube, tolerance) {
		guess = improveGuess(guess, num, improveCube)
	}

	return guess
}

func absoluteValue(num float64) float64 {
	if num < 0 {
		return -num
	}

	return num
}

func isGoodEnough(guess float64, num float64, f func(float64) float64, tolerance float64) bool {
	diff := f(guess) - num
	diff = absoluteValue(diff)

	return tolerance >= diff
}

func improveGuess(guess float64, num float64, f func(float64, float64) float64) float64 {
	guess = f(guess, num)

	return guess
}

func square(num float64) float64 {
	return num * num
}

func cube(num float64) float64 {
	return num * num * num
}

func improveSquare(guess float64, num float64) float64 {
	return (guess + num/guess) / 2.0
}

func improveCube(guess float64, num float64) float64 {
	return (2.0*guess + num/(guess*guess)) / 3.0
}

```

Now, squareRoot and cubicRoot functions are implemented in a much cleaner way and looks self-documenting, thanks to the small modules we made that do only one thing and do it very well. Introducing new root functions into the system does not take much effort not. 

Of course, we can still take this code a bit further by capturing the patterns in squareRoot and cubicRoot functions. But I will leave it to yourself <span class="emojify">&#x1F31D;</span>.


<br>

# Until the next time <span class="emojify">&#x1F60A;</span>




