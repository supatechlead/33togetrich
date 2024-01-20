+++ 
draft = false
date = 2024-01-20T14:37:18+01:00
title = "Nginx Reverse Proxy for Multiple React Apps"
description = "How to Dockerize Multiple React Apps on the Same Server"
slug = ""
authors = []
tags = ["Docker", "React", "Nginx", "Reverse Proxy"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

To set up a Docker container with Nginx as a reverse proxy for multiple React.js application containers on the same server, follow these steps:

## React Projects Creation

First, we have to create the 2 react projects we want to access via the reverse proxy.

```bash
npx create-react-app react-project-1
npx create-react-app react-project-2
```
Then, ensure that each React application is correctly running

## Create Container and Export ports

Each React container should expose a port on which the application is running. We can specify the exposed port in the `Dockerfile` or when running the container. 

Here's a suitable  `Dockerfile`:

```bash
FROM node:14-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

Here's example for running the container:
```bash
docker run -p 3000:3000 your_image_name
```

For each Dockerfile, we may have a `.dockerignore` file as:
```bash
*/node_modules/
*/build/
*/.git/
.gitignore
Dockerfile
.editorconfig
```

After `Dockerfile` and `.dockerignore` has been completed, we can build `react-image-1` and `react-image-2` with commands:
```bash
docker image build -t react-image-1 .
docker image build -t react-image-2 .
```
Then, we can  start `app1` and `àpp2` containers with:
```bash
docker run --name app1 react-image-1
docker run --name app2 react-image-2
```


## Create a Docker Network
Create a Docker network so that all containers can communicate with each other. This will also make it easier to configure the reverse proxy.

```bash
docker network create network1
```

## Configure Nginx as Reverse Proxy:

Create an Nginx configuration file defining proxy rules for each React application. Here is the `nginx.conf`, configuration file:

```bash
server {
    listen 80;
    server_name localhost

    location /app1 {
        proxy_pass http://app1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /app2 {
        proxy_pass http://app2:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Add more configurations for each React.js application
}
```
We also need a `mime.types` file that contains:

```bash
types {
    text/html                             html htm shtml;
    text/css                              css;
    text/xml                              xml;
    image/gif                             gif;
    image/jpeg                            jpeg jpg;
    application/javascript                js;
    application/atom+xml                  atom;
    application/rss+xml                   rss;

    text/mathml                           mml;
    text/plain                            txt;
    text/vnd.sun.j2me.app-descriptor      jad;
    text/vnd.wap.wml                      wml;
    text/x-component                      htc;

    image/png                             png;
    image/tiff                            tif tiff;
    image/vnd.wap.wbmp                    wbmp;
    image/x-icon                          ico;
    image/x-jng                           jng;
    image/x-ms-bmp                        bmp;
    image/svg+xml                         svg svgz;
    image/webp                            webp;

    application/font-woff                 woff;
    application/java-archive              jar war ear;
    application/json                      json;
    application/mac-binhex40              hqx;
    application/msword                    doc;
    application/pdf                       pdf;
    application/postscript                ps eps ai;
    application/rtf                       rtf;
    application/vnd.apple.mpegurl         m3u8;
    application/vnd.ms-excel              xls;
    application/vnd.ms-fontobject         eot;
    application/vnd.ms-powerpoint         ppt;
    application/vnd.wap.wmlc              wmlc;
    application/vnd.google-earth.kml+xml  kml;
    application/vnd.google-earth.kmz      kmz;
    application/x-7z-compressed           7z;
    application/x-cocoa                   cco;
    application/x-java-archive-diff       jardiff;
    application/x-java-jnlp-file          jnlp;
    application/x-makeself                run;
    application/x-perl                    pl pm;
    application/x-pilot                   prc pdb;
    application/x-rar-compressed          rar;
    application/x-redhat-package-manager  rpm;
    application/x-sea                     sea;
    application/x-shockwave-flash         swf;
    application/x-stuffit                 sit;
    application/x-tcl                     tcl tk;
    application/x-x509-ca-cert            der pem crt;
    application/x-xpinstall               xpi;
    application/xhtml+xml                 xhtml;
    application/xspf+xml                  xspf;
    application/zip                       zip;
}
```

## Run Nginx Container with Custom Configuration

Start the `Nginx` container by mounting the custom configuration file inside the container.

```bash
docker run -p 80:80 \
--network my_network \
-v "$(pwd)"/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v "$(pwd)"/nginx/mime.types:/etc/nginx/mime.types \
nginx:latest
```

## Test Your Configuration:

Access your React.js applications via your server's domain or IP address, followed by the path for each application.

For example, if your server is at IP 192.168.1.100 and you configured two React.js applications under paths /app1 and /app2, access them via http://192.168.1.100/app1 and http://192.168.1.100/app2.