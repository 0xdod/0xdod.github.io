+++
title = 'Deploying a Static Website on Kubernetes'
date = 2025-06-25T21:05:38+01:00
draft = true
tags = ['devops', 'kubernetes']
+++

As part of my commitment to building mastery in kubernetes, I'm starting with something simple but foundational: deploying a static website on Kubernetes.

It's quite an overkill and there is almost no practical reason for hosting a static site in kubernetes, but the goal here is to get hands on experience with Kubernetes components specifically pods, deployments and services, which I consider to be the building blocks of any Kubernetes application. 

## What We'll be Building

We'll deploy a static site served by [caddy]() on a local kubernetes cluster using [minikube](), at the end of this project, we should be able to tell:

- What a Pod is and how containers are run
- How a Deployment can be used to manage Pods- How to expose our application to the external world with Services

## Prerequisites

- [Docker]() or [Docker Desktop]()
- [minikube]() 
- [kubectl]() 
- [git]()
- docker hub account

The code for this project can be found in this [repo]()

## Prepare the Static Website

We are going to be using a custom website for this project, any static website would do but this one here is a free open source example site that we will use for this project. 

We have to build a docker image that bundles our site and server in one package and upload it to a container registry so our cluster can then pull from there.

We pull the example site from the repo or you can use a simple `index.html` file.

```bash
git clone https://github.com/cloudacademy/static-website-example.git
```

Since we will be using caddy server, we need to create a configuration to tell the server how to serve our files.

We create a new file called `Caddyfile` with the following content

```Caddyfile
:80
root * /srv
file_server
```

This tells the server to act as a file server, listening on port 80 and serving files in the /srv directory.

Then we create a `Dockerfile` which contains commands that will help build our image.

```Dockerfile
FROM caddy:2.7.5-alpine
WORKDIR /srv
COPY static-website-example /srv
COPY Caddyfile /etc/caddy/Caddyfile
EXPOSE 80
CMD ["sh", "-c", "caddy run --config /etc/caddy/Caddyfile --adapter caddyfile"]

```

We then build our image and push to dockerhub.

```bash
docker build -t k8s-static-website-example .
```

Before pushing to dockerhub we may need to authenticate
```bash
docker login
```

Follow the prompts to login and on successful login, push the newly built image to dockerhub

```bash
docker push <username>/k8s-static-website-example
```

With these out of the way, we can focus on kubernetes next.

## Pods

In kubernetes, pods are the smallest deployable units of compute. An application in a container runs in a pod. A pod can contain more than one container but it is usually recommended to run one container per pod. 

Containers running in a pod share the same IP address and network namespace. To learn more about pods, please refer to the official guide and docs.

To define a pod, you can use the following yaml file 


## Deployments 


## Services
