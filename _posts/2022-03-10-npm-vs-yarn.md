---
title: "NPM vs YARN, what is your favorite?"
categories:
  - Server
date: 2022-03-10 06:00:25 +0900
tags:
  - package install
  - npm
  - yarn
---

To make portfolio in Web service, I complete task as below.   
1.  Set portfolio frontend(React) and create Dockerfile
2.  Set Nginx configuration
3.  Set docker-compose. react-web(expose 3000), nginx(80:80)
4.  Associate AWS EC2 instance static IP with AWS-route-53 domain

I installed react packages with npm. However the speed was so slow.   
So, while looking for alternatives, there was the following data that comparatively analyzed **NPM** and **YARN**.



> #### Yarn vs. NPM: How to Choose?
> It's essential to consider the advantages and disadvantages of both NPM and Yarn when deciding which one to use.   
> ### [Yarn]
> **Advantages**   
> * Supports **parallel installation** and **Zero installs**, both of which dramatically increase performance.   
> * Newer versions of Yarn offer a more secure form of version locking.   
> * Active user community.
> 
> **Disadvantages**   
> * Yarn doesn't work with Node.js versions older than version 5.
> * Yarn has shown problems when trying to install native modules.
> 
> ### [NPM]
> 
> **Advantages**
> 
> * Easy to use, especially for developers used to the workflow of older versions.   
> * Local package installation is optimized to save hard drive space.   
> The simple UI helps reduce development time.   
> 
> **Disadvantages**    
> * The online NPM registry can become unreliable in case of performance issues. This also means that NPM requires network access to install packages from the registry.   
> * Despite a series of improvements across different versions, there are still security vulnerabilities when installing packages.   
> * Command output can be difficult to read.   
{: .prompt-info }


In summary, in terms of speed, npm processes package installs as a sequence, whereas yarn installs in parallel, so yarn has high performance in speed. In particular, the higher the number of packages installed, the higher the performance of yarn.


![des](../../assets/p/1/yvsn.png)

By using yarn, I can deploy my service within 5 minutes without cache. Thanks for parallel installation of yarn that is more faster for importing a large number of packages than npm.(npm took **5 hours** to install with no-cache)

Below is information about yarn.

|                       | Yarn / Yarn 2.0                                          | NPM                                                                |
| :-------------------: | -------------------------------------------------------- | ------------------------------------------------------------------ |
|     Zero Installs     | Uses a "**.pnp.cjs**" file to reinstall packages offline | Unsupported                                                        |
|  Usage of Workspaces  | Supported                                                | Supported                                                          |
|         Speed         | **Parallel** installation                                | **Sequential** installation                                        |
|    Remote Scripts     | Supported using the command "yarn dlx"                   | Supported using the command "npx"                                  |
|      Plug'n'Play      | Uses a ".pnp.cjs" file                                   | Unsupported                                                        |
|     License Check     | Checks each package download while installing            | Unsupported                                                        |
| Generating Lock Files | Automatically created as "yarn.lock"                     | Automatically created as "package-lock.json"                       |
|     Dependencies      | Installs using the "yarn" command                        | Installs dependencies one by one through the "npm install" command |



So, here is my final version of react Dockerfile.

```dockerfile
FROM node:18-alpine3.16
RUN mkdir /app
WORKDIR /app

ENV PATH /app/node_modules/.bin:$PATH 

COPY package.json /app/package.json
COPY yarn.lock /app/yarn.lock
RUN yarn install 
COPY . /app 

CMD ["/app/start.sh"]
COPY .env /app/.env

CMD ["npm", "start"]
```
{: file="/react-portfolio-website/client/Dockerfile"}


`/app/start.sh` is a shell script which make `.env` file to change react default port.

```yaml
#!/bin/sh
set -e

echo "make .env with PORT=$PORT"
echo "PORT=$PORT" > .env
```
{: file="/react-portfolio-website/client/start.sh"}

Also, since I manage it with docker, I also have to take care of the image size. Here is the issue of how to reduce image size by **60%**. [How to reduce docker image size?](https://github.com/yarnpkg/berry/discussions/3201#discussioncomment-1086179)

Additionally, COPY `.pnp,cjs` in Dockerfile can utilize Zero installation of Yarn.






