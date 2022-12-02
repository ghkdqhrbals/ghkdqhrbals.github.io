---
title: "Banking backend server with Golang(Done)"
categories:
  - 서버
  - Golang
date: 2022-04-23 15:00:25 +0900
tags:
  - golang
  - backend
---
# Progress

For my experience in golang-backend, I use below skills for develop my banking service.
> You can see the source code in my github. [**Banking backend server**](https://github.com/ghkdqhrbals/golang-backend-master)

## Skills

| Skills          | Purposes                                            |
| --------------- | --------------------------------------------------- |
| RDS(Postgresql) | storing User,accounts,balance info                  |
| migration       | auto-migration                                      |
| sqlc            | generate Golang interface from sql                  |
| git-workflow    | auto-deploy                                         |
| gin             | HTTP communication                                  |
| bcrypt          | safe way to store PW                                |
| Viper           | auto-server configuration setting                   |
| Gomock          | testing RDS                                         |
| Docker          | auto-setting env and ease for run                   |
| Kubernetes      | auto-scaling and managing pods(docker images)       |
| JWT or PASETO   | TOKEN based authentication(reduce session weight)   |
| JQ              | conversion JSON to txt                              |
| AWS             | get fixed public IP and for automation, maintenance |

## How to automatically deploy our service?
![Alt text](../../assets/p/2/deploy.jpg)
![example](../../assets/p/2/ci-cd-deploy-example.jpg)
## We use AWS with following service
![Alt text](../../assets/p/2/aws-cloud.jpg)
## How can we safely store user password in RDS?
![Alt text](../../assets/p/2//safe-password-storing.jpg)
## How can we handle multiple api request with asynchronous response?
![Alt text](../../assets/p/2//api-multi-thread.jpg)


## Update[v1.4.4]
* __Set Kubernetes Cluster__
1. Set aws-ath.yaml to access AWS-EKS(with granted user)
2. Set deployment.yaml to get image from AWS-ECR and run with 2 replica(pod)
3. Set issuer.yaml to issue TLS certificate
    * get certificate from 'letsencrypt' with domain 'api.hwangbogyumin.com'(free)
4. Set ingress.yaml with Nginx ingress controller
    * request -> api.hwangbogyumin.com
    * api.hwangbogyumin.com -> aws-route-53 my arn
    * aws-route-53 my arn -> nginx-ingress address
    * nginx-ingress address -> ingress-service(TLS)
    * ingress-service ->> server pods(1,2)

* __Use AWS-Route-53 to create Domain & Set Kubernetes Ingress-service pods__

## Update[v1.4.3] 
* __Use Git Action for auto AWS docker image upload__
1. Set Configure AWS credentials
2. Add AWS_ACCESS_KEY_ID, KEY in Github Repositry secrets
    * __AWS-IAM__ secrets:AWS_ACCESS_KEY_ID, AWS_ACCESS_KEY
3. Launch deploy action
    * Get secrets from Git and Access with token
    * Login
    * build images and Deploy to __AWS-ECR__ ap-northeast-2

* __Add services(AWS-ECR, AWS-Secrets Manager, AWS-IAM, AWS-RDS)__
1. Secrets Manager for managing symmetric_key that can encrypt/decrypt Paseto Payload and RDS port, RDS root, key
2. Set IAM(Identity and Access Management) for safe AWS resource access
3. Set ECR(Elastic Container Registry) in ap-northeast-2
4. Set RDS(Relational Database Storage) in us-west-1, postgres12

* __JQ__
1. Get RDS informations and etc. from AWS secrets manager
2. Transform AWS secrets format into JSON format using JQ
3. Based on json data, set app.env with corresponding data

## Update[v1.4.2]
* __Edit Dockerfile & Docker-compose file__
1. Set shell script(wait-for-it.sh) to wait until postgres is ready[Detail](https://github.com/peter-evans/docker-compose-healthcheck)
    * As we alpine image, 'apk add bash' needed
2. Set shell script(start.sh) to migrate db up
3. Edit Dockerfile to add needed files
    * migrate, app.env, main(object file), pre-setting shell script(wait-for-it.sh, start.sh)
4. Make docker-compose.yaml to specify services name and environment variables

## Update[v1.4.1]
* __Add Token Authentication Middleware__
1. Set user.go/loginUser for create/verify TOKEN
2. Set Route(createAccounts, transferMoney, etc.) Group that need authorization.
3. Make authMiddleware for pre-check requests whether they have TOKEN for authorization
3. Edit api/server.go
    * Before get request, check and verify http header's authorized part.
    * If there is a TOKEN that server created, pass request to actual handler.
    * If no TOKEN exists, abort session and send response.
4. 위의 http통신은 TLS로 encrypt되었음을 가정한다. __[TLS Details](https://github.com/ghkdqhrbals/simplebank/wiki/ghkdqhrbals:SSL-TLS)__
    * TLS가 적용되지 않았으면 TOKEN가 탈취되었을 때, Server에 권한없이 RPC 통신하여 DB 탐색가능.
* __Testcase정의__
```
1. User -----      Login       --> Server    [LoginParams] = username, password
2. User <----      TOKEN       --- Server    [TOKEN] = chacha20poly1305(nonce, Server's Key, AEAD, Payload{username, duration})
3. User -----    CreateAccount --> Server    [Params] = currency, TOKEN
4. User <----  Account's Info  --- Server    [Account] = verifyToken(Server's Key, TOKEN)
```
## Update[v1.4.0]
* __JWT(JSON Web Token)의 HMAC-SHA256(HS256) algorithm를 통한 payload+header 'Encryption' and 'MAC' 생성__
1. Set secretKey as random 256 bits(As we use HS256, Key should be 256 bits) Temporary!
2. Make CreateToken function(interface)
    * ( [HEADER]:'alg:HS256,typ:jwt', [PAYLOAD]:'id:string, name:string, expiredAt:time', [SIGNATURE]:'HMAC([HEADER],[PAYLOAD]).TAG' )
3. Make VerifyToken function(interface)
    * Check HEADER, SIGNATURE, ...
4. Set test enviroments
    * case Invalid Header algorithm, MAC failed, Expiration, etc.
* __PASETO(Platform-Agnostic Security Tokens)의 chacha20Poly1305 algorithm를 통한 payload+header+nonce 'Encryption' and 'MAC' 생성__
1. Set secretKey as random 256 bits(As we use chacha20Poly1305, Key should be 256 bits) Temporary!
2. Make CreateToken function(interface)
3. Make VerifyToken function(interface)
4. Set test env.

## Update[v1.3.1]
* __Set Testcase of managing User password__
1. Set api/user_test.go TestCreateUserAPI test function
    * cases: "OK", "InternalError", "DuplicateUsername", "InvalidUsername", "InvalidEmail", "TooShortPassword"
2. Set Custom reply matcher(gomock)

## Update[v1.3.0]
* __Use Bcrypt(Blowfish encryption algorithm) for safe storing user password__([Detail](https://github.com/ghkdqhrbals/simplebank/wiki/ghkdqhrbals:bcrypt))
1. Set util/password.go using bcrypt which can randomly generate cost, salt to get hashed password with params
2. Set util/password_test.go for testing 
3. Make api/user.go to set createUser handler
4. Set routes("/user") for request from clients

## Update History
* __Use Gin framework to communicate RPC([Details](https://github.com/ghkdqhrbals/simplebank/wiki/ghkdqhrbals:gin))__
1. Set router, routes
2. Set various handler
3. Get http request
4. Use custom validator to check if it is a valid request.
5. Binding JSON to STRUCT(request)
6. Access Local Database -> Execute transactions -> Get results(all process can handle with error)
7. Response

* __Use Viper for auto configuration setting ([Details](https://github.com/ghkdqhrbals/simplebank/wiki/ghkdqhrbals:viper))__
1. Set /app.env
2. Set /util/config.go
3. import configurations in /main.go

* __Use Gomock to remove DB dependency from tests in service layer ([Details](https://github.com/ghkdqhrbals/simplebank/wiki/ghkdqhrbals:mockdb))__
1. Use sqlc interface with all query functions to interface
2. Edit /.bash_profile for PATH to go/bin(to using mockgen)
3. Execute mockgen to generate mock functions
4. __Set APIs for testing(TestGetAccountAPI)__