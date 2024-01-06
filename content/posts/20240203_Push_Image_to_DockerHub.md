+++ 
draft = false
date = 2024-01-03T14:37:18+01:00
title = "Push a Docker Image to DockerHub"
description = "A Guide for Pushing Docker Image to DockerHub"
slug = ""
authors = []
tags = ["DockerHub"]
categories = []
externalLink = ""
series = []
tableOfContents = true
+++

## Create a Repository on DockerHub
Before we can push an image to Docker Hub, we will first need an account on Docker Hub. The signup process is simple and available on the front page of Docker Hub.

Once we have an account, our next step will be to create a new repository for this article. This process is fairly self-explanatory, so we can create the named `push-example` repo.

## Building a New Image
As we create a `push-example` repo, we will reating an image named with convention `<username>/push-example`, where `<username>` is the personal user name for Docker Hub, and `push-example` the reference for the previously created repo. So, as my user name is `devopsjaz`, image would be built with:
```bash
docker build -t devopsjaz/push-example .
```
If `push-example` was built using `docker build -f push-example .`, we could rename its tag using:
```bash
docker image tag push-example devopsjaz/push-example
```

The reason for this is because Docker Hub organizes repositories by user name. Any repository created under any account includes its username in the Docker image name.

This same approach is used when building images for an organization's repository. 

## Pushing the Container

All we need to do is issue the docker push command specifying our image tag name.
```bash
docker push devopsjaz/push-example
```