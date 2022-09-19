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
> They both can manage cpu-usage, memory-usage, etc.

-------------------

# **Docker-compose**
  * Docker Compose is a tool for defining and running multi-container Docker applications used in **single host**!
  > I usally set my docker configuration with docker-compose. I use this in (1) portfolio project, (2) golang-backend project. My projects have features below. **You can check docker-compose setting in my other posting**.
  > ![a](../../assets/p/6/2.png)
  > ![a](../../assets/p/6/1.png)

```yaml
services:
  postgres:
    image: postgres:12-alpine
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=simple_bank
    ports:
      - "5432:5432" # purpose to exposeing ports. Except this ports, you can use only inside services
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - DB_SOURCE=postgresql://root:secret@postgres:5432/simple_bank?sslmode=disable
    depends_on:
      - postgres
    entrypoint: ["/app/wait-for-it.sh","postgres:5432","--","/app/start.sh"]
    command: ["/app/main"]
```
{: file='golang-backend/docker-compose.yaml'}

```yaml
version: "3.7"

services:
  nginx:
    restart: always
    container_name: nginx
    build:
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "80:80"
    networks:
      - frontend
      
  client:
    container_name: client
    expose:
      - "3000"
    restart: "on-failure"
    environment:
      - PORT=3000
      - NODE_ENV=development 
      - CHOKIDAR_USEPOLLING=true
    build:
      context: ./client
      dockerfile: Dockerfile
    volumes:
      - "./client/:/app"
      - "/app/node_modules"
    stdin_open: true
    networks:
      - frontend

networks: 
  frontend:
    driver: bridge
```
{: file='portfolio/docker-compose.yaml'}

-------------------

# **Kubernetes**
  * Kubernetes is a platform for managing containerized workloads and services, that facilitates both declarative configuration and automation
  > if you run your service in single host, you don't need to use kubernetes. But if you want to run your service in multiple-host and take leverage in automation, you can use Kubernetes for your convenience.

<!-- ![a](../../assets/p/6/k8s.png) -->
![a](../../assets/p/6/kubernetes_detail.png)

Now, I will explain each components.

## Components

* `Cluster`   
: `Cluster` is a set of `Controll Plane` and one or more `Worker Node`.
  * `Controll Plane`   
  : also called `Master Node`. And it manages the `Worker Node`s and the `Pod`s in the cluster.
    * `API server`   
    : entry point for REST/kubectl
    * `Scheduler`   
    : schedules pods to worker nodes
    * `Controll Manager`   
    : it manages and watches their current state of `Worker Node`, `Pod`
    * `etcd`(key-value store)    
    : stores all of Kubernetes cluster data(cluster state and config)
  * `Worker Node`
  :maintain running `Pod` and provide the Kubernetes runtime environment
    * `kubelet`    
    : It makes sure that containers are running in a Pod and they are healthy    
    : Path between API server of `Controll Plane`
    * `kube-proxy`     
    : manages IP translation and routing    
    : It facilitating Kubernetes networking services and load-balancing across all pods in a service
    * `Container runtime`       
    : It pulls images from `Container Registry` and starts and stops containers
    > `Container Registry` : can be `Docker Hub`, `Amazon Elastic Container Registry(ECR)`, `Google Container Registry(GCR)`

## Types of yaml used in Kubernetes

> To run kubernetes, we need to set configuration files with yaml format. There are various type. `Deployment`, `Service`, `Ingress`, `ClusterIssuer`, etc.
{: .prompt-info}


### Deployment
: A `deployment` type is responsible for keeping a set of pods running. Here is an example of deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: golang-backend-api-deployment
  labels:
    app: golang-backend-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: golang-backend-api
  template:
    metadata:
      labels:
        app: golang-backend-api
    spec:
      containers:
      - name: golang-backend-api
        image: ghkdqhrbals/simplebank:latest
        imagePullPolicy: Always
        ports:
          - containerPort: 8080
        env:
          - name: DB_SOURCE
            value: postgresql://root:secret@postgres:5432/simple_bank?sslmode=disable
```
{: file='deployment.yaml'}

* apiVersion : set api version. Here is a organized API information[https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html)
* kind : this is a type of configuration
* metadata : store resource label, name
* spec : details of components
  * replicas : set the number of `pod`
  * selector : what will deployment want to replicate(**find in `template`**)
  * template.spec.container :    
    (1) find `ghkdqhrbals/simplebank:latest` Docker images from **Docker Hub**,     
    (2) run container with name `golang-backend-api`,     
    (3) set container port `8080`

### Service
:A `service` is responsible for **enabling network access to a set of pods**.
> when you use only service, **need to create pods passively**


```yaml
apiVersion: v1
kind: Service
metadata:
  name: golang-backend-api-service
spec:
  type: ClusterIP #diff. LoadBalancer, etc.
  selector:
    app: golang-backend-api
  ports:
    - protocol: TCP
      # nodePort is external access port outside the cluster. But, as we set type as clusterIP, this setting isn't needed
      # nodePort: 30131
      port: 80 # internal port
      targetPort: 8080 # forward port
```
{: file='service.yaml'}

> reference from [https://matthewpalmer.net/kubernetes-app-developer/articles/service-kubernetes-example-tutorial.html](https://matthewpalmer.net/kubernetes-app-developer/articles/service-kubernetes-example-tutorial.html)

* spec.type : you may choose within `ClusterIP` or `LoadBalancer` or `NodePort`
  * ClusterIP : The service is only accessible from within the Kubernetes cluster
  > you can't make requests to your `Pod` from outside the cluster
  * NodePort : The service can handle requests that originate from outside the cluster
  * LoadBalancer : The service becomes accessible externally through a cloud provider's load balancer functionality

In this case, (1) service get request from port:80 internally, (2) select pods with labeled `golang-backend-api`, (3) forward request to container port `8080` in golang-backend-api pod.


### Ingress
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

![a](../../assets/p/6/ingress.png)


```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: golang-backend-api-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  ingressClassName: nginx
  rules:
  - host: "api.hwangbogyumin.com"
    http:
      paths:
      - pathType: Prefix # 443, 80 etc. -> 80 if "/" prefix
        path: "/"
        backend:
          service:
            name: golang-backend-api-service
            port:
              number: 80
  tls:
  - hosts:
    - api.hwangbogyumin.com
    secretName: hwangbogyumin-api.cert
```
{: file='ingress.yaml'}


**~On proceeding...~**
> **I have been already finished this kubernetes works in AWS, not in on-premise. So, here i will set configuration in local environment**








# References
* [https://www.upguard.com/blog/docker-vs-vmware-how-do-they-stack-up](https://www.upguard.com/blog/docker-vs-vmware-how-do-they-stack-up)
* [https://stackoverflow.com/questions/47536536/whats-the-difference-between-docker-compose-and-kubernetes](https://stackoverflow.com/questions/47536536/whats-the-difference-between-docker-compose-and-kubernetes)
* [https://github.com/compose-spec/compose-spec/blob/master/spec.md](https://github.com/compose-spec/compose-spec/blob/master/spec.md)
* [https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/What-is-Kubernetes-vs-Docker-Compose-How-these-DevOps-tools-compare](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/What-is-Kubernetes-vs-Docker-Compose-How-these-DevOps-tools-compare)
* [**https://medium.com/devops-mojo/kubernetes-architecture-overview-introduction-to-k8s-architecture-and-understanding-k8s-cluster-components-90e11eb34ccd**](https://medium.com/devops-mojo/kubernetes-architecture-overview-introduction-to-k8s-architecture-and-understanding-k8s-cluster-components-90e11eb34ccd)
* [https://matthewpalmer.net/kubernetes-app-developer/articles/service-kubernetes-example-tutorial.html](https://matthewpalmer.net/kubernetes-app-developer/articles/service-kubernetes-example-tutorial.html)
* [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)