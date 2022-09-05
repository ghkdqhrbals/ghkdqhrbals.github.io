---
title: "Saga Architecture Pattern"
categories:
  - Server
  - Saga
date: 2022-09-04 16:00:25 +0900
tags:
  - saga
---

# What is Saga Pattern?

* SAGA pattern is a pattern that guarantees **Atomicity** in a distributed environment by exchanging events between **microservices** and **sourcing a reward event** to microservices that have completed previous work when an operation in a specific microservice fails.
  
* Why did this pattern arise?
  * As rise of MSA(that user information is in DB_1, DB_2, DB_3 ...), ACID transactions become impossible
  * Need to manage transactions for distributed services
* Saga has 2 patterns
  * **Choreography-based Saga**
  * **Orchestration-based Saga**

## 1. Choreography-based Saga Pattern
![choreography](../../assets/p/5/choreography.jpeg)
* Each services **manage** their own local transaction
* And within service, it determine which service to send canceled transaction events to
> The Choreography-based Saga pattern **manages local transactions within the service it has**, and when the transaction ends, a completion event is issued. If there is a transaction to be performed next, an event is sent to the service that needs to perform the transaction, and the service receives the completion event and proceeds with the next operation. Do this sequentially.
> 
> Events can then be delivered asynchronously through message queues such as Kafka.


## 2. Orchestration-based Saga Pattern
![Orchest](../../assets/p/5/centeral.jpeg)
* Each services **have** their own local transaction
* However, composite service manages all transcations sequentially.
* And composite service determine which service to send canceled transaction events to

> In my opinion, It is necessary to avoid the transaction in a separate space by setting the DB separately. But if you can't avoid that, also when you doing with your project with MSA, i do recommand choreography-based saga pattern. **Since MSA is an architecture adopted for independent services, i think you should keep the key words which is independent**. With Choreography, **you can mange your services independently**. **As your services become larger and larger, Choreography makes your management of transaction easier than Orchestration**.
{: .prompt-info}



# References
* [https://waspro.tistory.com/735](https://waspro.tistory.com/735)
* [https://microservices.io/patterns/data/saga.html](https://microservices.io/patterns/data/saga.html)
* [https://stackoverflow.com/questions/4127241/orchestration-vs-choreography](https://stackoverflow.com/questions/4127241/orchestration-vs-choreography)