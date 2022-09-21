---
title: "Goroutine"
categories:
  - Server
  - Golang
date: 2022-09-18 15:00:25 +0900
tags:
  - thread
  - goroutine
---
Golang use goroutine which is similar with thread, but little bit different. Goroutine can make you easy to use thread with concurrency. Also goroutine makes your program faster and lighter than when using original OS thread.

Basically, goroutine and OS thread are same in how they share their resoruces(Heap, Data, Code section of memory) in process

> * **Code section**
>   * Store whole binary code
> * **Data section**
>    * Store Global variable, static variable, array, structure
> * **Heap section**
>   * With malloc & free or new & delete, structure or variable are allocated & return
> * Stack section
>   * Store local variable, argument variable when function called
{: .prompt-info}

Now, here is a difference of OS thread and goroutine.

# Goroutine
Goroutine is M:N thread

![goroutine](../../assets/p/6/goroutine.png)
* `G` (Goroutine) : Goroutine
  * has Stack Pointer, Program Counter, DX
* `M`(Machine) : OS thread
  * has pointer of `P`
  * run `G` by `P`'s LRQ
* `P`(Processor) : logical processor
  * has one `LRQ`
  * allocate `G` in `M`
  * if System Call that block thread occured in `G`, re-allocate `G` into left `M`
  > to show block is never happened
* `LRQ`(Local run queue) : Run Queue
  * `P` pop `G` from its `LRQ` and allocate to `M`
  * every P has its own `LRQ`
* `GRQ`(Global run queue) : Run Queue
  * if every `LRQ` full, `G` is stored in `GRQ`




### Memory usage
* goroutine
  * need **2KB stack**
  > as `G` only need SP, PC, DX
* thread(Java)
  * need more than **1MB stack**
 
### Context Switch
* goroutine
  * as goroutine is 2KB, **context switch cost is very cheap**
  * Also as context switch is very easy, **concurrency performance is higher** than original OS thread
* thread(JAVA)
  * context switch cost is expensive
  * Java use OS thread as their abstract thread, concurrency performance is lower than 




# References
* [https://syntaxsugar.tistory.com/entry/GoGolang-Scheduler](https://syntaxsugar.tistory.com/entry/GoGolang-Scheduler)
* [https://velog.io/@kineo2k/고루틴은-어떻게-스케줄링되는가](https://velog.io/@kineo2k/고루틴은-어떻게-스케줄링되는가)
* [https://go.dev/src/runtime/HACKING](https://go.dev/src/runtime/HACKING)