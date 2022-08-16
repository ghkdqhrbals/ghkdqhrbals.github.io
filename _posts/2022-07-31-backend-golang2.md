---
title: "Golang fullstack(2)"
categories:
  - Server
date: 2022-07-31 15:00:25 +0900
tags:
  - Golang
  - fullstack
  - reverse proxy
  - backend server
  - frontend server
  - Micro Service Architecture
---

고정 IP할당을 받기 위해 EC2 인스턴스를 생성하려 하였지만, 지난번 비용이 과하게 지출되어 기존의 EC2 인스턴스를 사용하기로 하였다.

기존의 EC2 인스턴스는 현재 프트폴리오 Website를 배포중에 있다.

Docker로 같은 네트워크로 엮어놔서 port 번호만 달리 하면 통신 가능하다.

따라서 해당 nginx configuration file을 수정하여 backend를 위한 패스를 설정할 것이다.

```conf

user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {                     
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    upstream docker-client {
        server client:3000;
    }
    server {
        listen 80;
        server_name portfolio.hwangbogyumin.com;

        location /sockjs-node {
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_pass http://docker-client;
        }

        location / {
            proxy_pass         http://docker-client;
                proxy_redirect     off;
                proxy_set_header   Host $host;
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        }

    }
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
                                                
    sendfile        on;                                                                         
    keepalive_timeout  65;                                                                      
    include /etc/nginx/conf.d/*.conf;           
}

```


URL = http://portfolio.hwangbogyumin.com

```
Client - Nginx
            |-- Portfolio WAS "/" -> "localhost:3000"
            |-- banking  "localhost:3100"
                  |-- banking api "/api"
                        |-- user api "/api/user"
                        |-- transfer api "/api/transfer"
                        |-- account api "/api/account"
                        |-- etc.
```

1. banking에서 paseto TOKEN을 발급하며, 이를 통해 세션 유지 부담을 줄인다.
2. Postgresql에 blowfish encryption으로 user 패스워드 정보를 안전하게 저장한다.
3. 자동화를 위한 Dockerlizing
4. CI/CD를 git actions로 자동관리설정하였다(with AWS).
   * AWS EKS, ECR 비용이 상당히 많이 나왔다. 한달 가량 사용하였을 때, 20만원 가까이 지출이 발생하여 깜짝 놀랐다...
5. Middleware은 authentication을 위한 TOKEN creation & validation 기능을 수행한다.
   * 해당 부분은 server이 많이 질 수록 고민이 많아져야한다. TOKEN이 가지는 단점이 분명히 존재하기 때문이다. 이는 추후에 다룰 것이다.(ex. TOKEN 탈취 시나리오, 동시 접속 미허용시 핸들링 미비)


필자가 원하는 다음 단계는 다음과 같다.

```
Client - Nginx - frontend server
                  |--- banking backend Server...
```

조금더 상세하게 표현자면 다음과 같다.

```
Client - Nginx(80:3000) - frontend server(internal 3000)
                            |--- banking backend Server(internal 3100)
```

즉, front server를 추가하고, 외부에 80(+443:아직 TLS 미적용)을 노출한다.
front는 backend server와 http통신할 것이다(서버 내부통신임으로 암호프로토콜 미적용).

모든 부분을 dockerlize할 것이며, CI/CD는 추후 적용할 것이다.

이를 위한 frontend를 react를 활용하여 작성을 시작한다.