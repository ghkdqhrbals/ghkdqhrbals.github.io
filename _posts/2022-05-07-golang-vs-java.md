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
## What is Golang and Java?
### Golang
Golang is an open-source language from Google that made in 2011.

The syntax of Golang is close to "C" because the language's compiler was built in C. But it is now written in Golang, allowing the language to remain self-hosted.

Golang is a concurrent programming language designed for modern multicore processors, meaning it can do numerous tasks at once. It also features deferred garbage collection, which manages memory to run programs quickly.

### Java
Java is a statically typed general-purpose programming language. Sun Microsystems developed and released Java in 1995. 

Java used to be the language of choice for server-side applications, but it no longer holds that position. Despite that, hundreds of various applications around the world employ it. Various platforms, ranging from old legacy software on servers to modern data science and machine learning applications, use Java. 

There are ample pre-built modules and codes available because Java is famous among developers. These modules and developer availability make coding in Java easy.

Java is versatile. It runs anywhere there's a processor. It's similar to a compiled language where the virtual machine breaks down the code into bytecode before compiling it. 

### Golang vs Java

|                    | Golang                                   | Java               |
| ------------------ | ---------------------------------------- | ------------------ |
| Type Hierarchy     | Cannot                                   | Can(OOP structure) |
| Performance        | High(Non-virtual machine)                | Low(JVM)           |
| Community          | Small                                    | Large              |
| Concurrency        | Powerful                                 | less-Powerful      |
| Garbage Collection | Powerful                                 | less-Powerful      |
| run on             | OS(so, performance high, light weighted) | JVM()              |

Both Java and Golang are powerful, popular, and useful. But still, they have significant differences.    
Java is **1. object-oriented(OOP)**, and has a **2. larger library and community**.   
Golang is **1. better supports GC(Garbage Collection)** and **2. better supports concurrency**, **3. light-weight**.

> **While Golang runs faster than Java, Java has more features and better support**.
{: .prompt-info}

## Difference in Garbage Collection
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

In Java, the equivalent expression would be to make Rect and Point classes, in which case the Min and Max fields in Rect would be pointers to **separately allocated objects**.
> This requires more allocated objects, taking up more memory, and giving the garbage collector more to track and more to do. On the other hand, it does avoid ever needing to create a pointer to the middle of an object.

Compared to Java, then, Go gives you the programmer more control over memory layout, and you can use that control to **reduce the load on the garbage collector**. That can be very important in programs with large amounts of data. 

> So, if you want to remove Rect from memory, just delete start address with four integers size. This avoids memory fragmentation and allocates memory at the end of the heap when new objects are created, **allowing for fast memory allocation and deletion**. I think that as memory is bigger and bigger, this trend would be fitted.
{: .prompt-info}

Control over memory layout may also be important for extracting performance from the hardware due to cache effects and such, but that's tangential to the original question.

The collector in the current Go distributions is reasonable but by no means state of the art. We have plans to spend more effort improving it over the next year or two. To be clear, Go's garbage collector is certainly not as good as modern Java garbage collectors, but we believe it is easier in Go to write programs that don't need as much garbage collection to begin with, so the net effect can still be that garbage collection is less of an issue in a Go program than in an equivalent Java program.

### Static GC(Golang) vs Dynamic GC(Java)
Golang **do not re-arrange objects in heap when they remove some of objects**. And that is one of the static GC's characteristic. Thus, static GC has issue that memory fragmentations occurs.
![static gc](../../assets/p/3/static_GC.png)
> To handle that fragmentation issues, Golang use **TCMalloc**(Thread-Caching Malloc) for efficient memory management(for multi-thread programming)!
{: .prompt-info}

> #### What is TCMalloc?
> TCMalloc is Thread-Caching Malloc.
> 
> 1. TCMalloc **reduces lock contention for multi-threaded programs**.
> 
> TCMalloc is faster than the glibc 2.3 malloc (available as a separate library called ptmalloc2) and other mallocs. ptmalloc2 takes approximately 300 nanoseconds to execute a malloc/free pair on a 2.8 GHz P4 (for small objects).
> 
> Another benefit of TCMalloc is space-efficient representation of small objects.
> 
> **TCMalloc treates objects with size <= 32K** ("small" objects) differently from larger objects. **Large objects are allocated directly from the central heap** using a page-level allocator (a page is a 4K aligned region of memory). I.e., a large object is always page-aligned and occupies an integral number of pages.

### Java has Dynamic GC
![dynamic gc](../../assets/p/3/dynamic_GC.png)

* Also Java has adopted **aging system(Generational garbage collection)** for Garbage Collection.


> To summarize, Golang use TCMalloc to **reduce lock contention for multi-threaded programs**. So, as most server leverage multi-threaded programs, Golang can reduce server's memory(Golang dont need virtual machine) and reduce lock contention which means **Golang run faster than Java**.   
> 
> (In Korean) Java는 JVM 위에서 돌아가기 때문에 실행하기 위해선 byte코드를 machine코드로 변환하는 과정이 필요하다. 반면 golang은 빌드과정에서 이미 machine코드로 변환했기 때문에 바로 동작할 수 있다. 빌드에 걸리는 시간도 GO 언어 내부적으로 최적화를 많이 해둬서 빠른편이다. GC는 각기 다른 측면이 있기에 무엇이 낫다고 정할 수는 없지만, (1) 경량 스레드를 지원하며, (2) 스레드 별 cashing을 적극적으로 지원하는 Golang이 multi-threading 환경에서는 더 낫다고 보여진다. 즉, 비동기를 위한 multi-threading 환경이 적용된 server는 자신의 퍼포먼스를 증가하기 위해 Golang을 선택하는 것은 타당하다고 생각된다.
{: .prompt-info}



## Difference in Concurrency and Simplicity

* Java uses OS thread to perform parallel execution of work through green threads(threads managed by language runtime). Golang uses OS thread through goroutines. **So in the parallelism there can't be significant difference between both implementations**. 

* But in concurrency there is huge difference. In java JVM map its green threads to OS threads while Golang brings mapping **goroutines to OS threads into deep abstraction level through go scheduler(run in go-runtime, not virtual machine)**.

### Comparison of Concurrency programming with example
#### Java
```java
public static void main(string[] args){
  new FixedThreadPoolExecutor();
}
```
{: file='main.java'}

```java
package taskExecutor;

import java.util.concurrent.TimeUnit;

public class Task implements Runnable {
    private String name;

    public Task(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
    @Override
    public void run() {
        try{
            Long duration = (long)(Math.random()*10);
            System.out.println("Doing a task during : " + name);
            TimeUnit.SECONDS.sleep(duration);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
```
{: file='Task.java'}

```java
package taskExecutor;

import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

public class FixedThreadPoolExecutor {
    public FixedThreadPoolExecutor(){
        ThreadPoolExecutor executor = (ThreadPoolExecutor)Executors.newFixedThreadPool(4);
        for(int i=0; i<10;i++){
            Task task = new Task("Task" + i);
            System.out.println("A new task has been added: "+ task.getName());
            executor.execute(task);
        }
        System.out.println("Maximum threads inside pool " + executor.getMaximumPoolSize());
        executor.shutdown();
    }
}
```
{: file='FixedThreadPoolExecutor.java'}

#### Golang
* As you can see, Golang is more simple than Java. **With Golang, we can easly access IPC with channel.**


```go
package main

import (
	"runtime"
	"fmt"
)

func main() {
	nCPU:=runtime.NumCPU()
	runtime.GOMAXPROCS(nCPU)

	const maxNumber  = 100

	ch := make(chan int)
	defer close(ch)

	go Generate(ch)

	for i:=0; i<maxNumber;i++{
		prime := <-ch
		fmt.Println(prime)
		ch1:=make(chan int)
		go Filter(ch,ch1,prime)
		ch=ch1
	}
}

func Generate(ch chan<- int){
	for i:=2; ;i++ {
		ch <-i
	}
}

func Filter(in <-chan int,out chan <-int, prime int){
	for{
		i:= <-in
		if i%prime !=0{
			out <- i
		}
	}
}
```





#### Go Runtime vs Java Virtual Machine
Does Go have a runtime?

Go does have an extensive library, called the runtime, that is part of every Go program. The runtime library implements garbage collection, concurrency, stack management, and other critical features of the Go language. Although it is more central to the language, Go's runtime is analogous to libc, the C library.

> It is important to understand, however, that **Go's runtime does not include a virtual machine**, such as is provided by the Java runtime. Go programs are compiled ahead of time to native machine code (or JavaScript or WebAssembly, for some variant implementations). Thus, although the term is often used to describe the virtual environment in which a program runs, in Go the word "runtime" is just the name given to the library providing critical language services.
> 
> [reference from **https://go.dev/doc/faq#runtime**](https://go.dev/doc/faq#runtime)

# GCC, LLVM and JVM
[**Golang vs Java**](https://www.turing.com/blog/golang-vs-java-which-language-is-best/)    
Golang - LLVM(https://github.com/llir/llvm)