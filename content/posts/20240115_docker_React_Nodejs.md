+++ 
draft = false
date = 2024-01-15T14:37:18+01:00
title = "MySQL in Docker in More Advanded Ways"
description = "More Advanced MySQL inside a Docker Container"
slug = ""
authors = []
tags = ["Docker", "MySQL"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

Docker is among the more popular platforms for developing and deploying containerized applications. This article explains how to Dockerize MySQL for isolated database environments. 

## Starting Your MySQL Container

First identify the image tag you should use. MySQL versions 5.6, 5.7, and 8.0 are available.

Starting a MySQL container for the first time will automatically create an initial `root` user. You need to either supply a password for this user or ask MySQL to generate one. Here's an example of running a basic MySQL container with a specified `root` password:
```bash
docker run --name mysql -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=change-me \
    --restart unless-stopped \
    mysql:8
```

* This command starts a container with MySQL 8. 
* The password for the `root` user is set manually. 
* The `-d` flag means the container will run in the background until it's stopped, independently of your terminal session. 
* The `--restart` parameter instructs Docker to always restart the container. This means your MySQL database will run without intervention after host machine reboots or Docker daemon updates. 
* The `unless-stopped` policy used here won't start the container if you manually stopped it with docker stop.
* Docker's `-p` flag enables port forwarding into the container so you'll be able to access your database on `localhost:3306`. This is the default MySQL port; this example forwards port 3306 on your host to the same port inside the container.

You can view the container's startup logs with `docker logs mysql --follow`. When "`ready for connections`" appears, your MySQL database is accessible.

## Container Access

Without port forwarding enabled, you'd only be able to access your database from within the container. You can do this at any time by using `docker exec` to get a shell inside the container:
```bash
docker exec -it mysql mysql -p
```
This command runs `mysql -p` inside the mysql container. The `-it` flags mean your terminal's input stream will be forwarded to the container as an interactive TTY.

### Persisting Data With Volumes

While the container created above is a fully functioning MySQL server, you need to set up volumes so your data isn't lost when the container stops. 

The MySQL Docker image is configured to store all its data in the `/var/lib/mysql` directory. Mounting a volume to this directory will enable persistent data storage that outlives any single container instance.

Stop and remove your earlier container to avoid naming conflicts:
```bash
docker stop mysql
docker rm mysql
```

Then start a new container with the revised configuration:
```bash
docker run --name mysql -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=change-me \
    -v mysql:/var/lib/mysql \
    mysql:8
```
Using this command to start your MySQL container will create a new Docker volume called mysql. It'll be mounted into the container at `/var/lib/mysql`, where MySQL stores its data files. Any data written to this directory will now be transparently stored in the Docker-managed volume on your host.

## Using Container Networks

In the examples above, port forwarding was used to expose the MySQL server on your host's network. If you'll only be connecting to MySQL from within another Docker container, such as your API server, a better approach is to create a dedicated Docker network. This improves security by limiting your database's exposure.

First create a Docker network for your application:
```bash
docker network create example-app
```
Specify this network when starting your MySQL container:
```bash
docker run --name mysql -d \
    -e MYSQL_ROOT_PASSWORD=change-me \
    -v mysql:/var/lib/mysql \
    --network example-app \
    mysql:8
```

Connect another container to the same network:
```bash
docker run --name api-server -d \
    -p 80:80 \
    --network example-app \
    example-api-server:latest
```
Your API and MySQL containers now share a network. You can connect to MySQL from your API container by referencing the MySQL container's hostname. This matches the container's name by default. Here your application should connect to port 3306 on the mysql host.

## MySQL Configuration
The official MySQL image supports several environment variables that you can use to configure your container's initial state. You've already seen one, `MYSQL_ROOT_PASSWORD`. Use the `-e` flag with `docker run` to set each of these variables. They're only respected the first time the container starts, when the MySQL data directory is empty.
* `MYSQL_DATABASE` - The name of a database schema to be created when the container starts.
* `MYSQL_USER` and `MYSQL_PASSWORD` - Create a new ordinary user when the container starts.
* `MYSQL_RANDOM_ROOT_PASSWORD` - Set this instead of `MYSQL_ROOT_PASSWORD` if you'd like MySQL to generate a secure root password for you. If you enable this setting, the password will be emitted to the container(s logs (accessible via the `docker logs` command) during the first start. It will not be possible to retrieve the password afterward.
* `MYSQL_ALLOW_EMPTY_PASSWORD` - Setting this will create the `root` user with an empty password. Only use this option for throwaway database instances. It is insecure and would let anyone connect to MySQL with superuser privileges.

Using these environment variables means their values will be visible to anyone able to `docker inspect` your container. A more secure approach is to use Docker secrets or volumes to inject values as files.

The MySQL image supports an additional variant of each of the above variables. Suffix a variable's name with `_FILE` to have its value interpreted as a path to a file containing the real value. This example securely sets the root user's password in a way that can't be inspected from outside the container:
```bash
mkdir secrets
echo "P@$$w0rd" > secrets/mysql-root-password

docker run --name mysql -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql-root-password \
    -v ./secrets:/run/secrets \
    --restart unless-stopped \
    mysql:8
```
The password is written to a file that's mounted into the container using a Docker volume. MySQL instructs that the password be sourced from that mounted file by way of the `MYSQL_ROOT_PASSWORD_FILE` environment variable. Anyone viewing the container's environment variables will see the file path instead of the plain text password.

## Creating a Custom Image

It can be helpful to create your own Docker image if your app requires custom MySQL configuration. Adding extra layers atop the official MySQL base image gives you a ready-to-use image where you can omit manual injection of a MySQL config file.

Here's an example `my.cnf` that changes some MySQL settings:
```bash
innodb-ft-enable-stopword = 0
innodb-ft-min-token-size = 1
```

The MySQL image loads config files stored in the `/etc/mysql/conf.d` directory. Files will only be read when the MySQL server starts, which is when you start your Docker container. To get your config into your container, either use another Docker volume to bind mount your file, or use a `Dockerfile` to bake your changes into a new image:
```bash
Copy FROM mysql:8
COPY my.cnf /etc/mysql/conf.d/my.cnf
```
Build your image:
```bash
docker build -t custom-mysql:latest .
```

Now you can run your image to start a MySQL instance that automatically uses your config file:
```bash
docker run --name custom-mysql -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=change-me \
    -v mysql:/var/lib/mysql \
    custom-mysql:latest
```
Since your custom image is based on the official Docker Hub version, you can use all the existing environment variables described above.

