---
title: Girlfriend
date: 2018-09-16 01:37:25
categories:
 - golang
tags:
 - golang
 - girlfriend
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

本文并非正经博客，just a girlfriend.在本文，你将看到 go 语言是如何定义变量，类型，使用“继承”，其次还会看到如何将函数绑定到对象上，实现我们平时所说的面向对象编程中的 `.` 调用。当然因为不是正经博客，所以本文不会讲解语法，你看完只会收获一个女朋友（开心么？兴奋么？），没有任何讲解。

看看我们的最终结果：

``` 
I walk 520 steps to meet you.
I really want to tell you my name, but I have forgotten it.
it is hard to believe it?
don't say anything, just kiss me.
your arms are really warm.
event if I am a bunch of code, I have no entity, can't leave the screen.
but I will stay with you forever, I love you.
```
<!--more-->

the code here:

``` go
package main

import "fmt"

func main() {
	gf := &girlfriend{
		people: people{Name: "",
			HairColor: "black",
			Age:       18,
			Height:    160.00,
			Weight:    45.00},
		Sex: "female",
	}
	gf.walk(520)
	fmt.Printf("%v\n", gf)
	gf.say("it is hard to believe it?")
	gf.kiss()
	gf.hug()
	gf.confess()
}

type people struct {
	Name      string
	HairColor string
	Age       int
	Height    float64
	Weight    float64
}

type girlfriend struct {
	people
	Sex string
}

func (p *people) String() string {
	if p.Name == "" {
		return "I really want to tell you my name, but I have forgotten it."
	}
	return fmt.Sprintf("my name is %v", p.Name)
}

func (p *people) walk(n int) {
	fmt.Printf("I walk %v steps to meet you.\n", n)
}

func (p *people) say(s string) {
	fmt.Printf("%v\n", s)
}

func (g *girlfriend) kiss() {
	fmt.Println("don't say anything, just kiss me.")
}

func (g *girlfriend) hug() {
	fmt.Println("your arms are really warm.")
}

func (g *girlfriend) confess() {
	fmt.Println("event if I am a bunch of code, I have no entity, can't leave the screen.")
	fmt.Println("but I will stay with you forever, I love you.")
}
```
