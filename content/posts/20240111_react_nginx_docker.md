+++ 
draft = false
date = 2024-01-11T14:37:18+01:00
title = "Dockerize React apps with Nginx - 2nd Method"
description = "Create a Minimal Containerization for React and Nginx"
slug = ""
authors = []
tags = ["Docker", "React", "Nginx"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

Dockerizing a React application with Nginx is a common practice for deploying web applications in a containerized environment. Here is a second step-by-step guide on how to Dockerize a React application with Nginx.

## Step 1: Creating A React Project

Assuming you already have a React application, make sure it is functionnaly correct. A minimal React application can be created usind
```bash
npx create-react-app my-app
```
Then enter into your project directory
```bash
cd my-app
```
## Step 2;  Creating Docker Files


### Create Dockerfile and docker-compose.yml

First, we need to create a `nginx` directory that will contain the `nginx.conf` configuration file.
```bash
mkdir nginx
```
Then, we can crate `Dockerfile`, `docker-compose`, and `nginx.conf` files:
```bash 
touch Dockerfile docker-compose.yml nginx/nginx.conf
```

#### Dockerfile

Open `Dockerfile` and put the following content inside it:
```  bash
# build environment
FROM node:13.12.0-alpine as build
WORKDIR /app
COPY . .
RUN yarn
RUN yarn build

# production environment
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY --from=build /app/nginx/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Whats happening here ?
* We’re telling Docker to grab a copy of Node, specify its Linux distribution as Alpine and name it to build. Why Alpine? Alpine Linux is much smaller than most distribution base images (~5MB), and thus leads to much slimmer images in general.
* Setting our working directory to app
* Copying project to our directory
* Running yarn to install packages
* Running build script to generate build files
* Telling docker to grap nginx-alpine image
* Copying build files
* Copying nginx configuration files to replace the default configuration
* This line is just for documentation that our application will work on port 80
* Running nginx

#### nginx.conf

Open `nginx.conf` ans put the following content inside it:
```bash
server {
  listen       80;
  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html =404;
  }
}
```
We are just mentioning the position of our application static files to let Nginx consume them whenever someone sends a request to port 80.

#### docker-compose.ymls

Open `docker-compose.yml` ansd put the following content inside it:
```bash
version: "2"
services:
  nginx-react:
    container_name: ngixreactapp
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 80:80
    environment:
      NODE_ENV: production
```
We are giving our app a name, mentioning the dockerfile to use, mapping port 80 to the application port 80, adding some environment variables.


## Step 3: Run our Container

We can run our container using;
```bash
docker-compose up
```
Also running it in detached mode using:
```bash
docker-compose -d up
```
