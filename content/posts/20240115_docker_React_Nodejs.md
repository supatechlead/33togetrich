+++ 
draft = false
date = 2024-01-15T14:37:18+01:00
title = "NodeJS Express backend inside Docker"
description = "Dockerize NodeJS Backend Server with React Frontend"
slug = ""
authors = []
tags = ["Docker", "React", "Node", "Express"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

In this tutorial we will learn how to create and run a NodeJS Express backend and a React frontend inside of a Docker container.

## Project Creation

Before you begin make sure that you have Docker installed and running on your computer.

Now use the command line to navigate to a directory like your desktop then run the commands below.

```bash
mkdir my-app-docker
cd my-app-docker
touch docker-compose.yml
mkdir api
cd api
npm init -y
npm i express
touch app.js Dockerfile .dockerignore
cd ..
```
We setup a backend called api and created some Docker file.

## Backend Part

Put this in the `docker-compose.yml` file. Be careful with the yaml formatting otherwise you will get Docker errors when you try to run it.
```yaml
version: '3.8'
services:
  api:
    build: ./api
    container_name: api_backend
    ports:
      - '4000:4000'
    volumes:
      - ./api:/app
      - ./app/node_modules
```

Add this is the app.js file.
```javascript
const express = require('express');

const app = express();

const port = process.env.PORT || 4000;

app.get('/', (req, res) => {
  res.send('Home Route');
});

app.listen(port, () =>
  console.log(`Server running on port ${port}, http://localhost:${port}`)
);
```

Now add this line to the `.dockerignore` file.
```bash
node_modules
```

Next add this code to the `Dockerfile` file.
```bash
FROM node:16-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 4000

CMD ["node", "app.js"]
```

Lastly add this run script to the `package.json` file.
```json
"scripts": {

"start": "node app.js"

},
```
### Optional Nodemon Usage

Using `Nodemon` enables the server auto restart when changes occur

If you want to have the server restart every single time you make a change to the files in the backend then you can configure it to use `Nodemon`.

All you have to do is update the `Dockerfile` and `package.json` file inside of the `api` folder.

Update the code in the `Dockerfile` using the code below. We are now installing `Nodemon` at the start and using `dev` as the `run` command.

```bash
FROM node:16-alpine

RUN npm install -g nodemon

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 4000

CMD ["npm", "run", "dev"]
```

Now update the `package.json` file with this `run` script for `Nodemon`.
```json
"scripts": {

"start": "node app.js",

"dev": "nodemon -L app.js"

},
```
We just created a basic NodeJS Express app that runs on port 4000. That port is also mapped to 4000 in docker which lets us run it in a Docker container.

## Frontend Part

Now lets create a React frontend! Use your command line to get inside of the root folder for `my-app-docker`. Run the commands below to setup the project.
```bash
npx create-react-app client
cd client
touch .dockerignore Dockerfile
```

Add this line into the `.dockerignore` file.
```bash
node_modules
```

Put this code into the `Dockerfile` file.
```bash
FROM node:17-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```
## Assemble Backend and Frontend

Finally update the `docker-compose.yml` in the root folder with the code below. We have added a `client` section at the bottom with settings for getting React running inside of a Docker container. Be careful with the yaml formatting otherwise you will get Docker errors when you try to run it.
```yaml
version: '3.8'
services:
  api:
    build: ./api
    container_name: api_backend
    ports:
      - '4000:4000'
    volumes:
      - ./api:/app
      - ./app/node_modules
  client:
    build: ./client
    container_name: client_frontend
    ports:
      - '3000:3000'
    volumes:
      - ./client:/app
      - ./app/node_modules
    stdin_open: true
    tty: true
```

Getting the React app to run inside of Docker requires you to be in the root folder where the `docker-compose.yml` file is. Now run the command below and it should run inside of a Docker container.
```bash
docker-compose up
```
If you go to http://localhost:3000 you should see the home route in your browser window.

You can stop the server with the command below or you can go to the Docker app and stop the container from running.
```bash
docker-compose down
```
With this setup you can have a NodeJS backend and React frontend running at the same time inside of Docker! 