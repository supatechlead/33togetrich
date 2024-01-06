+++ 
draft = false
date = 2024-01-02T14:37:18+01:00
title = "Dockerize a React Application"
description = "A Step by Step Tutorial to Dockerize React Application"
slug = ""
authors = []
tags = ["Docker", "ReactJS"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

## Create a Simple React Application

Use the following command to set up a simple React application:
```bash
create-react-app react-docker-example
```
Run `npm start` to see if the app is running properly.

## Write a DockerFile

Let's go over the instructions we'll need, one by one.
```bash
FROM node:18-alpine
```

This instruction pulls the base image from a remote repository (in this case, the Docker Hub) and defines the starting point for the image layers. The syntax for specifying the image is `<image>:<tag>` where tag represents the version of the image.

```bash
WORKDIR /react-docker-example/
```
This command sets the working directory for any commands you add in the Dockerfile. So, while building the image, the commands will be executed in this directory.

```bash
COPY public/ /react-docker-example/public
COPY src/ /react-docker-example/src
COPY package.json /react-docker-example/
```
These instructions will copy the files we need into the working directory. We only need the public and src folders (where your code resides), and the package.json file to run the application.

```bash
RUN npm install
```
The `RUN` instruction executes any command by adding a new layer on top of the current ones, thus modifying the image. This modified image will be used for the next steps.

In this case, it installs all the dependencies specified in the `package.json` file. This is why we did not copy the `node_modules` folder into the working directory. The folder will be created after this command gets executed.

```bash
CMD ["npm", "start"]
```
This instruction defines the command that will be executed when starting a container from the image. There can only be one `CMD` instruction in the Dockerfile. If there are more than one, then only the last one will be considered.

Since `npm start` is the command used for starting a React app, we'll specify the same for running the container.

## Build the Image
Now that we have written the Dockerfile, it's time to build the image. Open your terminal and execute the following command.

```bash
docker image build -t <image_name>:<tag> <path>
```
`-t` option specifies the name and tag for the image. `<path>` represents the path in which you want to run the command.

We'll name the image `react-docker-example` and give it a tag `latest`. Make sure you change into the project's root directory using `cd react-docker-example` before executing the command.
```bash
docker image build -t react-example-image:latest .
```
The `.` at the end represents the current directory.

Once you hit enter, this command will execute each instruction in the Dockerfile one by one.

Now, run `docker images` to see a list of images in your system. You'll see the details of the image you just created.

## Create a Container from the Image  

At this point, we have bundled package including everything needed to run the React application. Now, we need to create a container to run your application. Run the following command.

```bash
docker run -dp 8000:3000 --name react-example-container react-docker-example:latest
```
* `-d` runs the container in detached mode – that is, it will run in the background and not display the running process on your terminal.
* `-p` maps the port in the form `<host_port>:<container_port>`. The host port represents the port on the host machine that is mapped to the port inside the container. Since a React app is exposed through port 3000, we will map it to the port 8000 on your host machine.
* The `--name` flag specifies the name for the container.
* After these, you specify the image name and tag. If the image does not exist on your local system, it will try to find the image in the Docker registry.

Now, run `docker ps` or `docker container ps` to show a list of all the running containers.

Now, open your browser and go to http://localhost:8000. You'll be able to access your application.