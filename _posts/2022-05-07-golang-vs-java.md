---
title: "Golang vs Java"
categories:
  - database
date: 2022-05-07 15:00:25 +0900
tags:
  - database
  - golang
  - postgres
---

[References1](http://goog-perftools.sourceforge.net/doc/tcmalloc.html)    
[References2](https://stackoverflow.com/questions/14322724/what-is-the-go-language-garbage-collection-approach-compared-to-others)     
[References3](https://groups.google.com/g/golang-nuts/c/m7IFRYnI-L4)     
[References4](https://go.dev/doc/faq#runtime)     
[References5](https://www.turing.com/blog/golang-vs-java-which-language-is-best/)    
[References6](https://velog.io/@kineo2k/Go-언어의-GC)     
[References7](https://engineering.linecorp.com/ko/blog/go-gc/)     

Recently, I had a job interview in Ebay. They generally use Java for their development. And my programming language is Golang. So, they asked me that **What is difference between Golang and Java?**. As a matter of fact, I never thought about that. I would like to take this opportunity to summarize the differences between Golang and Java.

# Golang vs Java
## What is Golang?
Golang is an open-source language from Google that made in 2011.

The syntax of Golang is close to "C" because the language's compiler was built in C. But it is now written in Golang, allowing the language to remain self-hosted.

Golang is a concurrent programming language designed for modern multicore processors, meaning it can do numerous tasks at once. It also features deferred garbage collection, which manages memory to run programs quickly.

## What is Java?
Java is a statically typed general-purpose programming language. Sun Microsystems developed and released Java in 1995. 

Java used to be the language of choice for server-side applications, but it no longer holds that position. Despite that, hundreds of various applications around the world employ it. Various platforms, ranging from old legacy software on servers to modern data science and machine learning applications, use Java. 

There are ample pre-built modules and codes available because Java is famous among developers. These modules and developer availability make coding in Java easy.

Java is versatile. It runs anywhere there's a processor. It's similar to a compiled language where the virtual machine breaks down the code into bytecode before compiling it. 

## Golang vs Java

|                    | Golang                    | Java               |
| ------------------ | ------------------------- | ------------------ |
| Type Hierarchy     | Cannot                    | Can(OOP structure) |
| Performance        | Fast(Non-virtual machine) | Slow(JVM)          |
| Community          | Small                     | Large              |
| Concurrency        | Powerful                  | less-Powerful      |
| Garbage Collection | Powerful                  | less-Powerful      |

Both Java and Golang are powerful, popular, and useful. But still, they have significant differences.    
Java is **1. object-oriented(OOP)**, and has a **2. larger library and community**.   
Golang is **1. better supports GC(Garbage Collection)** and **2. better supports concurrency**.
* While Golang runs faster than Java, Java has more features and better support.

## Golang GC and JVM GC
One advantage that we believe Go has over Java is that **it gives you more control over memory layout**. For example, a simple 2D graphics package might define:
```
type Rect struct {
    Min Point
    Max Point
}

type Point struct {
    X int
    Y int
}
```
In Go, a Rect is just **four integers contiguous in memory**. You can still pass &r.Max to function expecting a *Point, that's just a pointer into the middle of the Rect variable r.
> Golang doesn't 

In Java, the equivalent expression would be to make Rect and Point classes, in which case the Min and Max fields in Rect would be pointers to **separately allocated objects**.
> This requires more allocated objects, taking up more memory, and giving the garbage collector more to track and more to do. On the other hand, it does avoid ever needing to create a pointer to the middle of an object.

Compared to Java, then, Go gives you the programmer more control over memory layout, and you can use that control to **reduce the load on the garbage collector**. That can be very important in programs with large amounts of data. 

> So, if you want to remove Rect from memory, just delete start address with four integers size. This avoids memory fragmentation and allocates memory at the end of the heap when new objects are created, **allowing for fast memory allocation and deletion**. I think that as memory is bigger and bigger, this trend would be fitted.
{: .prompt-info}



Control over memory layout may also be important for extracting performance from the hardware due to cache effects and such, but that's tangential to the original question.

The collector in the current Go distributions is reasonable but by no means state of the art. We have plans to spend more effort improving it over the next year or two. To be clear, Go's garbage collector is certainly not as good as modern Java garbage collectors, but we believe it is easier in Go to write programs that don't need as much garbage collection to begin with, so the net effect can still be that garbage collection is less of an issue in a Go program than in an equivalent Java program.

### Golang has Static GC
Golang do not replace objects in heap. And that is one of the static GC's characteristic. However, static GC has issue that memory fragmentations occurs. 
![static gc](../../assets/p/3/static_GC.png)
> To handle that fragmentation issues, Golang use **TCMalloc**(Thread-Caching Malloc) for efficient memory management!
{: .prompt-info}

#### What is TCMalloc?


### Java has Dynamic GC
![dynamic gc](../../assets/p/3/dynamic_GC.png)

* Also Java has adopted **aging system** for Garbage Collection.

# Go Runtime vs Java Virtual Machine
Does Go have a runtime?

Go does have an extensive library, called the runtime, that is part of every Go program. The runtime library implements garbage collection, concurrency, stack management, and other critical features of the Go language. Although it is more central to the language, Go's runtime is analogous to libc, the C library.

> It is important to understand, however, that **Go's runtime does not include a virtual machine**, such as is provided by the Java runtime. Go programs are compiled ahead of time to native machine code (or JavaScript or WebAssembly, for some variant implementations). Thus, although the term is often used to describe the virtual environment in which a program runs, in Go the word "runtime" is just the name given to the library providing critical language services.
> 
> [reference from **https://go.dev/doc/faq#runtime**](https://go.dev/doc/faq#runtime)

# GCC, LLVM and JVM
[**Golang vs Java**](https://www.turing.com/blog/golang-vs-java-which-language-is-best/)


# Golang - LLVM(https://github.com/llir/llvm)