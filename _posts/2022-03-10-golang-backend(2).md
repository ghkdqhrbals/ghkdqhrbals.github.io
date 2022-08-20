---
title: "Docker Setting"
categories:
  - Docker
date: 2022-03-10 14:00:25 +0900
tags:
  - golang
  - backend
---

# Dockerfile

Below is my Banking Server backend Dockerfile. Here, I'm going to explain each steps.

```dockerfile
# Build stage
FROM golang:1.18.3-alpine3.16 AS builder
WORKDIR /app
COPY . .
RUN go build -o main main.go
RUN apk add curl
RUN curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz


# Run stage
FROM alpine:3.16
WORKDIR /app
COPY --from=builder /app/main .
COPY --from=builder /app/migrate ./migrate
RUN apk add --no-cache bash
COPY app.env .
COPY start.sh .
COPY wait-for-it.sh .
COPY db/migration ./migration


EXPOSE 8080
CMD ["/app/main"]
ENTRYPOINT ["/app/start.sh"]
```

Lets start from the beginning!

## Build Stage
* `FROM golang:1.18.3-alpine3.16 AS builder`   
First, alpine is small version of linux. They dont have bash, curl or other libraries. So for me, I need only golang environment and basic linux. Thus this compact version of linux fits to me.

* `WORKDIR /app`,`COPY . .`   
Set my local environment in `/app`.

* `RUN go build -o main main.go`   
run `go build -o main main.go` in docker container terminal. This command help me to get a compiled version of excutable file:main

* `RUN apk add curl`   
As I said before, in alpine version, there is no curl inside. So we need to install curl for download from github.

* `RUN curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz`    
I'm going to install golang-migrate to use `migrate` command tools for migration. So install and decompress files. the `-L` option means that redirect to location.

The purpose of this build stage is that literally building basic environments. Now its **Run Stage**.

## Build Stage
* `FROM alpine:3.16`    
  `WORKDIR /app`    
Basic Linux and Setting up.

* `COPY --from=builder /app/main .`   
In build stage, we will copy executable file main into `/app`

* `COPY --from=builder /app/migrate ./migrate`    
Also we will copy migrates(.exe) which is generated from `curl -L github.~~~`.

* `RUN apk add --no-cache bash`   
alpine image doesn't have bash, so I will install that. I will use bash for run shell scripts.

* `COPY app.env .`   
`app.env` contains server configuration like below. From this environment files, I use Viper library to automatically set configuration on server. But, only use Viper as local testing.

```yaml
DB_DRIVER=postgres
DB_SOURCE=postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable
SERVER_ADDRESS=0.0.0.0:8080
ACCESS_TOKEN_DURATION=15m
TOKEN_SYMMETRIC_KEY=12345678901234567890123456789012
```
{: file='app.env'}

* `COPY start.sh .`   
  I use `start.sh` to run db migration to my database and start the app.

```bash
#!/bin/sh
set -e
echo "run db migration"
/app/migrate -path /app/migration -database "$DB_SOURCE" -verbose up
echo "start the app"
exec "$@"
```
{: file='start.sh'}

* `COPY wait-for-it.sh .`   
  As I use two containers, Postgres and Backend-Service, my service needs to wait until Postgres container is ready. This shell script listen port 5432, so can decide whethere this port is ready. This file is long so you can check my github.

* `COPY db/migration ./migration`   
  `start.sh` generate SQL queries for migration in `db/migration`, and I will copy these schemes into `/migration`

* `EXPOSE 8080`   
  I will expose this service locally with port `8080`.

* `CMD ["/app/main"]`   
  Set `/app/main` as a default command.

* ENTRYPOINT ["/app/start.sh"]
  When this Dockerfile build & run in container, `/app/start.sh` will be executed **first**.

# docker-compose
* Docker compose simplifies management by automatically building and executing services from **multiple containers**. Below is my setting for composing docker containers.
  
```yml
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
{: file='docker-compose.yml'}

* `services:`    
  Declare that we are going to setting containers.
* `postgres:`   
  This service name & container name.
* `image: postgres:12-alpine`     
  We now get image of postgres from dockerhub.

* `environment: - POSTGRES_USER=root - POSTGRES_PASSWORD=secret - POSTGRES_DB=simple_bank`    
  Setting user, password, db name.

* `ports:`   
  Just for here, we will expose port outside.
* `api`, `build`, `context: .`, `dockerfile: Dockerfile`    
  We will build image with Dockerfile in this folder.
* `ports: - "8080:8080"`   
  Normally, backend server's port doesn't need to show their ports outside.

> Actually, It is critical when you expose Postgres ports outside. Because malicious users can access to server's DB and take out all informations. Also it is very vulnerable to DoS/DDoS attack(**especially low-rate DoS Attack**).
{: .prompt-danger }

* `environment: - DB_SOURCE=~~~`     
  Setting Env. Instead of `secret@localhost`, set as `secret@postgres` so postgres service's network will be fitted.   
  Generate services using docker compose will seperate `postgres` service and `api` service as a different IP address like `postgres`='172.16.0.**2**',`api`='172.16.0.**3**'. So `secret@localhost` means `secret@172.16.0.3`. But it should be `secret@172.16.0.2`. So rather set `secret@172.16.0.2`, you can just set a service name there.

* `depends_on: - postgres`    
  api service will need postgres service. So After postgres service is running, api service will run.

* `entrypoint: ["/app/wait-for-it.sh","postgres:5432","--","/app/start.sh"]`      
  First, run `/app/wait-for-it.sh`, so set listening port for postgres server.   
  Second, set `postgres:5432` as a argument of `wait-for-it.sh`.   
  Lastly, all things ready, start `start.sh`   

Before
```
gyuminhwangbo@Gyuminui-MacBookPro simplebank % docker images
REPOSITORY   TAG                 IMAGE ID       CREATED        SIZE
```
{: file='terminal'}

After
```
gyuminhwangbo@Gyuminui-MacBookPro simplebank % docker compose up
...
gyuminhwangbo@Gyuminui-MacBookPro simplebank % docker images
REPOSITORY       TAG                 IMAGE ID       CREATED        SIZE
simplebank_api   latest              7da6148aa0f0   3 weeks ago    52.1MB

gyuminhwangbo@Gyuminui-MacBookPro simplebank % docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS                    NAMES
de0dfecaf757   simplebank_api       "/app/wait-for-it.sh…"   47 seconds ago   Up 45 seconds   0.0.0.0:8080->8080/tcp   simplebank-api-1
5a9c72502355   postgres:12-alpine   "docker-entrypoint.s…"   47 seconds ago   Up 45 seconds   0.0.0.0:5432->5432/tcp   simplebank-postgres-1

gyuminhwangbo@Gyuminui-MacBookPro simplebank % docker network inspect simplebank_default
[
    {
        "Name": "simplebank_default",
        "Id": "8e834d593f880a08051e480628d94b0c61c91964a0e7d76c1e63ae249140193f",
        "Created": "2022-07-23T08:03:57.59455131Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "192.168.96.0/20",
                    "Gateway": "192.168.96.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "5a9c72502355f346543b02b0ba1a89563333f2c3d5450da25e68f6c8c32a14f9": {
                "Name": "simplebank-postgres-1",
                "EndpointID": "08fdd10da2e5d6ba2b5b9b7b69aebab0797c1e5fd2c24ce5a9a0f2be96aaa4e1",
                "MacAddress": "02:42:c0:a8:60:02",
                "IPv4Address": "192.168.96.2/20",
                "IPv6Address": ""
            },
            "de0dfecaf75713dbc30595e21cf978472919d6bce9101acba36ab49708fa53cd": {
                "Name": "simplebank-api-1",
                "EndpointID": "b969560e4d29bcf7cbfbbaf67d524215704919ae5720ce40befbbe7ebc19fb1a",
                "MacAddress": "02:42:c0:a8:60:03",
                "IPv4Address": "192.168.96.3/20",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "simplebank",
            "com.docker.compose.version": "2.4.1"
        }
    }
]
```

By sending HTTP request using Postman, results are below.

### Login

* Register `ghkdqhrbals`
![i](../../assets/p/2/register_non.png)

* Register again with `ghkdqhrbals` and here is violations!
![i](../../assets/p/2/register_violation.png)

* Register `Kim`
![i](../../assets/p/2/register.png)

* Login `Kim`
![i](../../assets/p/2/login.png)

* Both `ghkdqhrbals`, `Kim`'s register informations are successfully stored in postgres database.
![i](../../assets/p/2/db_table.png)
