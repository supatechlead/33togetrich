+++ 
draft = false
date = 2024-01-10T14:37:18+01:00
title = "Dockerize React apps with Nginx - 1st Method"
description = "Create a Minimal Containerization for React and Nginx"
slug = ""
authors = []
tags = ["Docker", "React", "Nginx"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

Dockerizing a React application with Nginx is a common practice for deploying web applications in a containerized environment. Here is a first step-by-step guide on how to Dockerize a React application with Nginx.

## Step 1: Set Up Your React App

Assuming you already have a React application, make sure it is functionnaly correct. A minimal React application can be created usind
```bash
npx create-react-app my-app
```

## Step 2: Create a Dockerfile

Create a `Dockerfile` in the root directory of your React app. This file will define how your application will be built and run within a Docker container.

```bash
# Use the official Node.js runtime as the base image
FROM node:18 as build

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the entire application code to the container
COPY . .

# Build the React app for production
RUN npm run build

# Use Nginx as the production server
FROM nginx:alpine

# Copy the built React app to Nginx's web server directory
COPY --from=build /app/build /usr/share/nginx/html

# Expose port 80 for the Nginx server
EXPOSE 80

# Start Nginx when the container runs
CMD ["nginx", "-g", "daemon off;"]
```

## Step 3: Create a .dockerignore File

Create a `.dockerignore` file in the same directory as your `Dockerfile` to exclude unnecessary files and directories from being copied into the Docker image.
``` bash
node_modules
build
npm-debug.log
```

## Step 4: Build the Docker Image

Now navigate to the root directory of your React application (where the `Dockerfile` is located), and build the Docker image:
```bash
docker build -t react-nginx-app .
```

This command will create a Docker image named `react-nginx-app`. 

## Step 5: Run the Docker Container

After building the Docker image, you can run the container:
```bash
docker run --name react-nginx-app -p 8080:80 -d react-nginx-app
```
This command will run the container in detached mode (`-d`) and map port 8080 on your host machine to port 80 inside the container.

## Step 6: Access Your React App

You can now access your React application by opening a web browser and navigating to http://localhost:8080 or the appropriate host and port you specified.

