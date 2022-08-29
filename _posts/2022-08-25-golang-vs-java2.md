---
title: "Golang vs Java(Part.2)"
categories:
  - Golang
date: 2022-08-25 15:00:25 +0900
tags:
  - golang
---
   

In concurrency, Golang and Java has multiple different features.

# Difference in Memory Usage, Cost of con/de-construction & Context Switch

Golang is known as a first-class support for concurrency. And it has standout ability to deal with multi-processing & multi-threading. So why they said like that? There are many advantages like below.
> Goroutine is light-weight thread of Golang.

### Memory Usage
* goroutine(Golang)
  * need only 2KB stack
  * can add heap storage if you need.
* thread(java)
  * need 1MB stack(500 times more than goroutine)
  * guard page needed
  * more thread, less heap available.

### Cost of construct/de-construct thread

* goroutine(Golang)
  * as memory usage is very small, cost of this is also small.
  * use thread pool
* thread(java)
  * use thread pool

### Cost of Context Switch

* goroutine(Golang)
  * low cost
  * save/restore with only 3 registers
  > PC(Program Counter), Stack Pointer, DX
* thread(java)
  * high cost
  * save/restore with 16 registers, etc.
  > etc : PC, SP, Segment Register, FP coprocessor state, AVX register, MSR, etc.

# Feature of goroutine

Go's mechanism for hosting goroutines is an implementation of what's called an M:N scheduler, **which means it maps M green threads to N OS threads**.

# References
* [https://betterprogramming.pub/deep-dive-into-concurrency-of-go-93002344d37b](https://betterprogramming.pub/deep-dive-into-concurrency-of-go-93002344d37b)