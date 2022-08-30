---
title: "Golang vs Spring Native"
categories:
  - Server
  - Comparison
  - Golang
date: 2022-08-30 16:00:25 +0900
tags:
  - golang
---

As I'm a golang developer, there is a article that compare between golang and spring.
> [Spring Boot Native vs Go: A performance comparison](https://ignaciosuay.com/spring-boot-native-vs-go-a-performance-comparison/)

The article said "**Golang is 133% faster than Spring**". They run various test to compare performance and it is clear to know the difference in performance! So, I would like to summarize that article in this post.

The test ramps up 200 users during the first minute and then keeps constantly 200 users for 2 hours. Spring native server is constructed with RESTFUL architectures.
> GET /products: Returns the last 20 products.   
> GET /products/{id}: Returns one single product for a given Id.   
> POST /products: Saves a new product.   


* The load test tries to simulate a common use case. Each user will perform the following actions:
1. Get all the latest products.
2. Save a new product.
3. Retrieve the product saved on step 2.

The test increments 200 users for the first minute and then maintains 200 users continuously for 2 hours.
# Test Results
## Response Time Results

> ### Spring Native
> ![SpringNativeResponseTimes](../../assets/p/3/SpringNativeResponseTimes.png)

---------------------------------------

> ### Golang
> ![GoResponseTimes](../../assets/p/3/GoResponseTimes.png)

## Resource Usage Results

> ### Spring Native
>![SpringPerformance](../../assets/p/3/Sprin2Performance-1.png)

---------------------------------------

> ### Golang
> ![GoPerformance](../../assets/p/3/GoPerformance.png)

> To summarize this test results, Golang's response time is **133%** faster than Spring-Native's. CPU and memory usage is also quite impressing. Golang use their memory less than 2%, while Spring Native use their memory more than 12%. Which means that **the performance of Golang is better than Spring Native**(although Goalgn doesn't have Generics).
{: .prompt-info}

# References
* [https://ignaciosuay.com/spring-boot-native-vs-go-a-performance-comparison/](https://ignaciosuay.com/spring-boot-native-vs-go-a-performance-comparison/)