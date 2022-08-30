---
title: "JWT(Json Web Token)"
categories:
  - Server
  - Session/Token
date: 2022-05-01 20:00:25 +0900
tags:
  - token
---

We generally use TOKEN for authentications. But we don't actually know the details about token. Today I will talk about token, espectially JWT(json web token).

First we should know about the **cookie**.

# Cookie

Cookie is a database which is stored in client's web browser by server. Server can store data up to 4KB(this is different among the browsers). To maintain the state(as http is stateless protocol), cookie can be used as a temporary state. Also cookie can reduce resource comsumption of server.

This cookie data is key:value pair set. Each cookie data is stored according to its domain.

# Session

Session is a data that stored in server(while cookie is stored in client). This session is normally used for authentication. To simply show how the session works, I will give you an good example here.
> User wants to login and get the current balance

1. User request(header:`GET /login`,body:`ID, Password`) to Server
2. Server check `main_DB` whether if users'ID and Password is correct
3. Server create `| session_id | username | expiration | ... |` data into server's `session_DB`
4. Server response with `session_id` to user
5. User browser's cookie get `session_id` and store into its cookie
6. User request(header:`GET /user/balance`,`Authorization:{session_id}`,body:`ID, Password`)
7. Server check `session_DB` whether if session_DB has `session_id` now
8. Server load `balance` from `main_DB`
9. Server response with `balance`

This is how session works. However, as more users connect to server and request `balance` at the same time, **the load on the session DB of the server increases**. You can simply scale up `session_DB`, but the cost is very expensive.

To reduce the load of session_DB when server service high-volume user environment, TOKEN is emerged!





쿠키는 서버가 클라이언트의 웹 브라우저에 저장하는 정보/파일 입니다. 최대 4kb까지 저장이 가능하며 쿠키의 최대 갯수는 브라우저마다 상이합니다. 상태를 유지를 포함해 여러 활용이 가능하며 클라이언트에게 파일을 저장하게 하므로 서버의 부하를 덜어주는 역할도 합니다.

쿠키는 key:value쌍을 가지며 두 값 외에도 여러 값을 가집니다. 아래는 go언어에서의 cookie 타입입니다. 몇 개 알아보자면 Path는 쿠키를 전송할 요청 경로를, Domain은 쿠키를 전송할 도메인을 뜻하며 Expires, MaxAge는 쿠키의 만료에 대한 요소인데 뒤에서 다뤄보겠습니다.