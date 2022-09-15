---
title: "Relationship of MSA & Docker"
categories:
  - Server
  - Docker
date: 2022-05-30 15:00:25 +0900
tags:
  - msa
---

MSA rises as **cloud services** are more and more used. That's because each service is independent and can be managed its cost efficiently. Also, even without the benefits of cost in cloud service, MSA has advantage of scalability, fast dev., increased team autonomy etc.

As each service of MSA has their independent environments, **(1) Consistent environmental management(Portability)** is required. Docker solves this for you!

> * VM : complicate   
> for example, just in case you want to upgrade the version of your program. With VM, you need to pull from registry, update all configuration files, reset environments, etc. So it is very hard to change or update your program    
> * Docker : easy    
> Docker can simplify this process. Download image, run it!

Also Docker can gives you more advantages. Docker can run your application **(2) fast in Provisioning & starting**, **(3) light-weight** than VM. This is because Docker based on OS-level process isolation rather than hardware virtualization(VM).


> ![a](../../assets/p/6/Docker_VM.png)
> containers provide OS-level process isolation whereas virtual machines offer isolation at the hardware abstraction layer (i.e., hardware virtualization).  So in IaaS use cases machine virtualization is an ideal fit, while containers are best suited for packaging/shipping portable and modular software    
> reference : [**https://www.upguard.com/blog/docker-vs-vmware-how-do-they-stack-up**](https://www.upguard.com/blog/docker-vs-vmware-how-do-they-stack-up)
{: .prompt-info}

> for example, if you want to input 'hellow world' in your Golang program.
> 
> * **VM** first seperate hardware space in your Host and install GuestOS(which is very big size). Next install Golang compiler, run program, I/O execution. At this time, there is transition between GuestOS and HostOS by its I/O driver.    
> * **Docker** only install in container with bin/lib files that needed for executing program(unlike VM need whole OS which is very big size). That means Docker has light-weight. Also container and HostOS share some parts of kernel which don't need I/O translation. Thus, speed of Docker is more faster than VM.   
> 
> Details in my posting : '**Docker vs Virtual Machine**'

To summary, Docker has many advatages. But if you have many many containers, how can we manage all these containers?

Here is two container orchestration technology options, Docker-compose and Kubernetes.

* Docker-compose
  * Docker Compose is a tool for defining and running multi-container Docker applications used in **single host**!

* Kubernetes
  * Kubernetes is a platform for managing containerized workloads and services, that facilitates both declarative configuration and automation
  > if you run your service in single host, you don't need to use kubernetes. But if you want to run your service in multiple-host and take leverage in automation, you can use Kubernetes for your convenience.

# Reference
[https://www.upguard.com/blog/docker-vs-vmware-how-do-they-stack-up](https://www.upguard.com/blog/docker-vs-vmware-how-do-they-stack-up)