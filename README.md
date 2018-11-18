# Docker: Best Practices

> Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package. 


## About me

 - Andrii Chyzh
 - Team Leader / Senior Software Engineer @ Wix.com
 
 
## Reference

 - [The Twelve-Factors Apps](https://12factor.net/)
 - [Docker Recommendation](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)


## Image size

### Use base images with minimal size
 
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              latest              4879790bd60d        2 days ago          101MB
ubuntu              latest              ea4c82dcd15a        4 weeks ago         85.8MB
centos              latest              75835a67d134        5 weeks ago         200MB
alpine              latest              196d12cf6ab1        2 months ago        4.41MB
``` 

### Don’t install unnecessary packages and remove not required

```dockerfile
RUN apt-get -y install curl && \
    curl https://test.com/file.json && \
    apt-get remove -y --purge curl
```

### Sort multi-line arguments

```dockerfile
RUN apt-get -y install \
    gnupg2 \
    lsb-release \
    software-properties-common \
    wget
```

### Use `.dockerignore` in every project, where you are building Docker images

```
.git
.gitignore
LICENSE
VERSION
README.md
Changelog.md
Makefile
docker-compose.yml
docs
```

### Create own base images for yours company use cases

```
FROM company-name/bootstrap-onbuild:latest
```

### Minimize the number of layers (instructions `RUN`, `COPY`, `ADD` creates layers) and use `multi-stage` builds


#### Case: [OpenResty](https://openresty.org/en/) (Nginx + Lua)

> OpenResty® is a full-fledged web platform that integrates the standard Nginx core, LuaJIT, many carefully written Lua libraries, lots of high quality 3rd-party Nginx modules, and most of their external dependencies. It is designed to help developers easily build scalable web applications, web services, and dynamic web gateways.


##### Regular usage

```dockerfile
FROM debian:stretch-slim

RUN apt-get -y update
RUN apt-get -y install gnupg2 lsb-release software-properties-common wget

RUN wget -qO - https://openresty.org/package/pubkey.gpg | apt-key add -
RUN add-apt-repository -y "deb http://openresty.org/package/debian $(lsb_release -sc) openresty"

RUN apt-get update
RUN apt-get -y install openresty

RUN apt-get remove -y --purge gnupg2 lsb-release software-properties-common wget
RUN apt-get -y autoremove
RUN rm -rf /var/lib/apt/lists/*

ENV PATH="${PATH}:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin"

COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

EXPOSE 80

CMD ["/usr/bin/openresty", "-g", "daemon off;"]
```

##### Base optimization

````dockerfile
FROM debian:stretch-slim

RUN apt-get -y update && \
    apt-get -y install gnupg2 lsb-release software-properties-common wget && \
    wget -qO - https://openresty.org/package/pubkey.gpg | apt-key add - && \
    add-apt-repository -y "deb http://openresty.org/package/debian $(lsb_release -sc) openresty" && \
    apt-get update && \
    apt-get -y install openresty && \
    apt-get remove -y --purge gnupg2 lsb-release software-properties-common wget && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="${PATH}:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin"

COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

EXPOSE 80

CMD ["/usr/bin/openresty", "-g", "daemon off;"]
````

##### Multi-stage builds (from `Docker 17.05`)

```dockerfile
# install stage
FROM debian:stretch-slim AS installer

RUN apt-get -y update && \
    apt-get -y install gnupg2 lsb-release software-properties-common wget && \
    wget -qO - https://openresty.org/package/pubkey.gpg | apt-key add - && \
    add-apt-repository -y "deb http://openresty.org/package/debian $(lsb_release -sc) openresty" && \
    apt-get update && \
    apt-get -y install openresty && \
    apt-get remove -y --purge gnupg2 lsb-release software-properties-common wget && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="${PATH}:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin"

# production stage
FROM debian:stretch-slim

COPY --from=installer /usr/local/openresty/ /usr/local/openresty/
COPY --from=installer /usr/bin/openresty /usr/bin/openresty

COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

EXPOSE 80

CMD ["/usr/bin/openresty", "-g", "daemon off;"]
```

#### Result

```
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
openresty           regular             146ddc05beb7        2 minutes ago        293MB
openresty           optimization        31a17c1644e3        About a minute ago   132MB
openresty           multistage          1eaace3ca6d8        13 seconds ago       65.2MB
```


#### Case: [Go](https://golang.org/) Application

> Go is an open source programming language that makes it easy to build simple, reliable, and efficient software.


##### Regular usage

```dockerfile
FROM golang:alpine
WORKDIR /app
ADD . /app
RUN cd /app && go build -o goapp
ENTRYPOINT ./goapp
```

##### Multistage builds (from `Docker 17.05`)

```dockerfile
# build stage
FROM golang:alpine AS builder
ADD . /src
RUN cd /src && go build -o goapp

# production stage
FROM alpine
WORKDIR /app
COPY --from=builder /src/goapp /app/
ENTRYPOINT ./goapp
```

#### Result

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
goapp               regular             1e97ed6a6d3a        20 seconds ago      316MB
goapp               multistage          bb6ce614ba3d        13 seconds ago      11MB
```


### Start / stop


### Configuration 


### Logging


### Docker Compose




## Link

![Link to repo](static/code.png)
