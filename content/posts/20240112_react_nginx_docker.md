+++ 
draft = false
date = 2024-01-11T14:37:18+01:00
title = "MySQL in Docker"
description = "Start using MySQL inside a Docker Container"
slug = ""
authors = []
tags = ["Docker", "React", "Nginx"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

Using MySQL in a Docker container can be beneficial for portability, ease of deployment, and resource efficiency. Here is a step by step guide for starting MySQL dockerization.

## Docker Installation Verification

First of all, you are sure that Docker was installed. Open terminal  and check the version with the following command:
```bash
docker version
```

## Download the Images

Next what it is necessary to do is download the images of MySql.
```bash
docker pull mysql/mysql-server
```
With this command, MySQL images are downloaded with the tag latest. Previous command can be extended as:
```bash
docker pull mysql/mysql-server:<tag>
```

## Start Instance of MySQL

Now if your images completed is, you can start the container. To start, you have to execute the following command, which consists of: 

* `docker run` the command for starting the docker container from the image
* `--name='my_sql_container'` after the argument, is defined the name of the container, without this argument name will be generated automatically
* `-d` a very important argument, which ensures, that the container in detached mode will be running
* `-p 3306:3306` mapping port from a docker container to host port. Because MySQL is running on port 3306 I mapped exactly this port outside of container.
* `mysql/mysql-server` name of our images

So the complete command is:
``` bash
docker run --name='my_sql_container' -d -p 3306:3306 mysql/mysql-server
```
We can check the correct Mysql starting using something as `docker ps` command

## Enter MySQL Container

First of all, we have to check the log because we need to get the initial password. Execute the command:
```bash
docker logs my_sql_container
```
Here is an example of what you might find in the logs:
```
GENERATED ROOT PASSWORD: A1bCd2EfGhI3
```
We are now able to enter MySQL container using:
```bash
docker exec -it my_sql_container bash
```
* `docker exec` execution docker command
* `-it` allows tty and keep stdin
* `my_sql_container` name of running container
* `bash` because we want to start bash

Once we are inside the container, we must go to the location of MySQL in the container using 
```bash
cd /var/lib/mysql 
```
then connect to the database with 
```bash
mysql -u root -p
```
and enter the initial password.

For the next work, it is also necessary to restart default password with
```SQL
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword'; 
```
Of course, you can choose any password.
Now we are ready to check the list of users:
```SQL
use mysql;
select user from user;
```

### Create a user for connection

In the last step, you have to create a user for the connection. I have chosen the name `dbeaver` with the same password but you are fully free with choice.
```SQL
CREATE USER 'dbeaver'@'%' IDENTIFIED BY 'dbeaver';
GRANT ALL PRIVILEGES ON *.* TO 'dbeaver'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
Finally, you are ready to connect with some database client. I have chosen for DBeaver. In the next picture, you can see the configuration. 
![DBeaver Configuration](/images/20240112_dbeaver_mysql.webp)
