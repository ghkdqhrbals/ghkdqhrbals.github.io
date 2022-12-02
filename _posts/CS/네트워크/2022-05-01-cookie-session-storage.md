---
title: "Cookie and Session, JWT(Json Web Token)"
categories:
  - CS
  - 네트워크
date: 2022-05-01 20:00:25 +0900
tags:
  - token
---

We generally use TOKEN for authentications. But we don't actually know the details about token. Today I will talk about token, espectially JWT(json web token).

First we should know about the **cookie**.

# Cookie

Cookie is a database which is stored in client's web browser by server. Server can store data up to 4KB(this is different among the browsers). To maintain the state(**as http is stateless protocol**), cookie can be used as a temporary state! Also cookie can reduce resource comsumption of server.

This cookie data is key:value pair set. Each cookie data is stored according to its domain.

# Session

Session is a data that stored in server(while cookie is stored in client). This session is normally used for authentication. To simply show how the session works, I will give you an good example here.
> User wants to login and get the current balance

![sessionDB](../../assets/p/3/sessiondb.png)

1. User request(header:`GET /login`,body:`ID, Password`) to Server
2. Server check `main_DB` whether if users'ID and Password is correct
3. Server create `| session_id | username | expiration | ... |` data into server's `session_DB`
4. Server response with `session_id` to user
5. User browser's cookie get `session_id` and store into its cookie
6. User request(header:`GET /user/balance`,`Authorization:{session_id}`)
7. Server check `session_DB` whether if session_DB has `session_id` now
8. Server load `balance` from `main_DB`
9. Server response with `balance`

This is how session works. However, as more users connect to server and request `balance` at the same time, **the load on the session DB of the server increases**. You can simply scale up `session_DB`, but the cost is very expensive.

**To reduce the load of session_DB when server service high-volume user environment, TOKEN is emerged!**

![token](../../assets/p/3/sessiondb2.png)

1. User request(header:`GET /login`,body:`ID, Password`) to Server
2. Server check `main_DB` whether if users'ID and Password is correct
3. Server create `token` data into server's `session_DB`
  > ![tokenConfiguration](../../assets/p/3/jwtGen.png)
  > It is important that you should never include personal information like `password` into your token.   
  > Because, JWT is basically encrypted with based64, which means that **everyone who have this token can look data inside**.   
  * Signature : **HASH**(`header`, `payload`, {`server_secret_key`})
4. Server response with `token` to user
5. User browser's cookie get `token` and store into its cookie
6. User request(header:`GET /user/balance`,`Authorization:{token}`)
7. Server validate `token` ---> this is a difference between session management and token
8. Server load `balance` from `main_DB`
9. Server response with `balance`

This is how token works in login example.

> **Main difference between token and session is that token doesn't need to maintain `session_DB`**
{: .prompt-info}

# Disadvantage of token

It seems that token based authentication is very simple and low cost. However, there is some disadvantages.

* When if token is stolen?   
  * if token is stolen by the others, you cannot restrict authentication process. But with the session DB, you can easily stop authentication process by removing session_DB's row.   
  * Also, session can inform you how many users with same id/pw are currently login.
> Thus, to prove endpoint, SSL/TLS is essential for http + token because they encrypt http & token and by doing that preventing from man in the middle attack.
{: .prompt-info}

* some signing algorithm are vulnerable
  * RSA PKCSv1.5 : padding oracle attack
  * ECDSA : invaild curve attack
* Set "alg" header to "HS256" while server verify token with RSA public key(RS256)
  * HASH(header,
  * you must!! check the "alg" in header

