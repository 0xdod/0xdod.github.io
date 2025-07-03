+++
title = 'Deploying a Static Website to a Kubernetes Cluster'
date = 2025-06-26T21:05:38+01:00
draft = true
tags = ['devops', 'kubernetes']
+++

> _Part 1 of the **10 Kubernetes Projects in 10 Weeks** challenge_

Welcome to the first of our Kubernetes project series! This week, we're diving into the fundamental building blocks of Kubernetes by deploying a simple static website using Caddy, a powerful web server, on our Minikube cluster.

The goal of this project is to solidify your understanding of core Kubernetes objects: **Pods**, **ReplicaSets**, **Deployments**, and **Services**. By the end, you'll see how these components work together to ensure your application is running reliably and is accessible.

## Prerequisites

- [Docker]() or [Docker Desktop]()
- [minikube]() 
- [kubectl]() 
- [git]()
- docker hub account

The code for this project can be found in this [repo]()

## The Static Site: Our Simple Web Page

For this project, we'll use a very basic `index.html` file. Imagine this is your personal portfolio, a blog, or any simple informational site.

```
git clone https://github.com/cloudacademy/static-website-example.git
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Awesome Static Site</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
            background-color: #f0f0f0;
            color: #333;
        }
        h1 {
            color: #007bff;
        }
        p {
            font-size: 1.2em;
        }
    </style>
</head>
<body>
    <h1>Welcome to My Kubernetes-Powered Static Site!</h1>
    <p>This page is served by Caddy, running inside a Kubernetes Pod.</p>
    <p>Exploring Pods, ReplicaSets, Deployments, and Services.</p>
</body>
</html>
```

We'll need to get this HTML file into our Caddy container. A common approach is to build a custom Docker image that includes your static content and a Caddy configuration. For simplicity, we'll assume Caddy is configured to serve files from its default `/srv` directory, and we'll place our `index.html` there.


## Caddy: Our Web Server

Caddy is an open-source, HTTP/2-enabled web server with automatic HTTPS. It's known for its simplicity and ease of configuration, making it a great choice for serving static sites.

We create a new file called `Caddyfile` with the following content

```Caddyfile
:80
root * /srv
file_server
```

Then we create a `Dockerfile` which contains commands that will help build our image.

```Dockerfile
FROM caddy:2.7.5-alpine
WORKDIR /srv
COPY static-website-example /srv
COPY Caddyfile /etc/caddy/Caddyfile
EXPOSE 80
CMD ["sh", "-c", "caddy run --config /etc/caddy/Caddyfile --adapter caddyfile"]
```

The next step is to build and push this image to a container registry, we would be using dockerhub to host our container repository so kubernetes can pull from there.


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


## Core Kubernetes Objects Explained

Before we deploy, let's briefly review the Kubernetes objects we'll be using:

### 1. Pods

- **What it is:** The smallest deployable unit in Kubernetes. A Pod is an abstraction over a container (or a group of tightly coupled containers) and represents a single instance of a running process in your cluster.
    
- **Key characteristics:**
    
    - Pods are ephemeral: they are not designed to be persistent. If a Pod dies, it's not automatically restarted.
        
    - They share the same network namespace, IP address, and storage volumes.
        
- **Analogy:** A Pod is like a single, self-contained worker unit in a factory, performing a specific task.
    

### 2. ReplicaSets

- **What it is:** A controller that ensures a specified number of Pod replicas are running at all times. If a Pod fails, the ReplicaSet automatically creates a new one to replace it.
    
- **Key characteristics:**
    
    - Ensures high availability by maintaining a desired number of Pods.
        
    - Primarily used for scaling and self-healing.
        
- **Analogy:** A ReplicaSet is like a supervisor in the factory, making sure there are always enough worker units (Pods) to meet the production quota.
    

### 3. Deployments

- **What it is:** A higher-level abstraction that manages the deployment and scaling of a set of Pods. Deployments use ReplicaSets under the hood to achieve their goals. They provide declarative updates to Pods and ReplicaSets.
    
- **Key characteristics:**
    
    - Handles rolling updates and rollbacks.
        
    - Manages the lifecycle of your application's Pods.
        
    - The most common way to deploy stateless applications.
        
- **Analogy:** A Deployment is like the factory manager who decides how many production lines (ReplicaSets) are needed and how to update them without stopping production.
    

### 4. Services

- **What it is:** An abstract way to expose an application running on a set of Pods as a network service. Services provide a stable IP address and DNS name for your application, even as Pods come and go.
    
- **Key characteristics:**
    
    - Decouples the application from the Pods' ephemeral nature.
        
    - Enables load balancing across multiple Pods.
        
    - Different types: `ClusterIP` (internal), `NodePort` (external via node IP), `LoadBalancer` (external via cloud load balancer), `ExternalName`.
        
- **Analogy:** A Service is like the front desk of the factory, providing a single, consistent address for customers to reach the factory, regardless of which specific worker unit (Pod) handles their request.
    

## Deploying Our Static Site with Caddy

We'll define two Kubernetes resources: a Deployment for our Caddy server and a Service to expose it.

### 1. Caddy Deployment (`caddy-deployment.yaml`)

This Deployment will manage our Caddy Pods. We'll use a public Caddy image and configure it to serve our static content. For simplicity, we'll use a `ConfigMap` to provide the `index.html` and Caddyfile.

First, let's create a ConfigMap for our static content and Caddyfile.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: caddy-config
data:
  index.html: |
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>My Awesome Static Site</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                text-align: center;
                margin-top: 50px;
                background-color: #f0f0f0;
                color: #333;
            }
            h1 {
                color: #007bff;
            }
            p {
                font-size: 1.2em;
            }
        </style>
    </head>
    <body>
        <h1>Welcome to My Kubernetes-Powered Static Site!</h1>
        <p>This page is served by Caddy, running inside a Kubernetes Pod.</p>
        <p>Exploring Pods, ReplicaSets, Deployments, and Services.</p>
    </body>
    </html>
  Caddyfile: |
    :80 {
        root * /usr/share/caddy
        file_server
    }
```

Now, the Deployment that uses this ConfigMap:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: caddy-static-site-deployment
  labels:
    app: caddy-static-site
spec:
  replicas: 2 # We want two replicas for high availability
  selector:
    matchLabels:
      app: caddy-static-site
  template:
    metadata:
      labels:
        app: caddy-static-site
    spec:
      containers:
        - name: caddy
          image: caddy:2.7.5-alpine # Using a lightweight Caddy image
          ports:
            - containerPort: 80 # Caddy listens on port 80 by default
          volumeMounts:
            - name: caddy-config-volume
              mountPath: /usr/share/caddy/index.html # Mount index.html
              subPath: index.html
            - name: caddy-config-volume
              mountPath: /etc/caddy/Caddyfile # Mount Caddyfile
              subPath: Caddyfile
      volumes:
        - name: caddy-config-volume
          configMap:
            name: caddy-config # Reference the ConfigMap we created
```

**Explanation:**

- `replicas: 2`: We're telling Kubernetes to maintain two Pods for our Caddy server. If one fails, the ReplicaSet (managed by the Deployment) will ensure a new one is spun up.
    
- `image: caddy:2.7.5-alpine`: Specifies the Caddy Docker image.
    
- `volumeMounts` and `volumes`: We're using a `ConfigMap` named `caddy-config` to inject our `index.html` and `Caddyfile` into the Caddy container. This is a clean way to manage configuration and static content for simple cases.
    

### 2. Caddy Service (`caddy-service.yaml`)

This Service will expose our Caddy Deployment to the outside world.

```
apiVersion: v1
kind: Service
metadata:
  name: caddy-static-site-service
spec:
  selector:
    app: caddy-static-site # Selects Pods with this label
  type: NodePort # Expose the service via a NodePort
  ports:
    - protocol: TCP
      port: 80 # Service port
      targetPort: 80 # Container port
      nodePort: 30080 # Accessible on your host at port 30080
```

**Explanation:**

- `selector: app: caddy-static-site`: This tells the Service to route traffic to any Pods that have the label `app: caddy-static-site`.
    
- `type: NodePort`: This makes our service accessible from outside the cluster using the IP address of any node and a specific port (the `nodePort`).
    
- `nodePort: 30080`: We've chosen `30080` for easy access from your host machine.
    

## Steps to Deploy

1. **Save the YAML files:**
    
    - Save the `ConfigMap` definition into `caddy-configmap.yaml`.
        
    - Save the `Deployment` definition into `caddy-deployment.yaml`.
        
    - Save the `Service` definition into `caddy-service.yaml`.
        
2. **Apply the ConfigMap:**
    
    ```
    kubectl apply -f caddy-configmap.yaml
    ```
    
    Verify: `kubectl get configmap caddy-config`
    
3. **Apply the Deployment:**
    
    ```
    kubectl apply -f caddy-deployment.yaml
    ```
    
    Monitor the deployment and underlying ReplicaSet and Pods:
    
    ```
    kubectl get deployment caddy-static-site-deployment
    kubectl get replicaset -l app=caddy-static-site
    kubectl get pods -l app=caddy-static-site -w
    ```
    
    You should see two Pods being created and eventually reach `Running` status.
    
4. **Apply the Service:**
    
    ```
    kubectl apply -f caddy-service.yaml
    ```
    
    Verify: `kubectl get service caddy-static-site-service`
    

## Verification

Once all resources are deployed and the Pods are running, you can access your static site:

1. **Get the Minikube IP:**
    
    ```
    minikube ip
    ```
    
2. **Access your site:** Open your web browser and navigate to `http://<minikube-ip>:30080`.
    

You should see your "Welcome to My Kubernetes-Powered Static Site!" page.

**Experimentation:**

- **Pod Resiliency:** Delete one of the Caddy Pods (`kubectl delete pod <pod-name>`). Observe how the ReplicaSet immediately creates a new one to maintain the desired replica count.
    
- **Scaling:** Scale your deployment up or down:
    
    ```
    kubectl scale deployment caddy-static-site-deployment --replicas=3
    kubectl get pods -l app=caddy-static-site
    ```
    
    Then scale back to 2.
    
- **Deployment Rollout:** Imagine you want to update your `index.html`. You would update the `ConfigMap` and then trigger a rollout of the Deployment (e.g., by changing the image tag, even if it's the same, or by using `kubectl rollout restart deployment caddy-static-site-deployment`). Observe the rolling update process where old Pods are gracefully terminated and new ones are brought up.
    

## Conclusion

This week, you've successfully deployed a static website on Kubernetes using Caddy, gaining hands-on experience with the foundational Kubernetes objects:

- **Pods:** The smallest unit of deployment.
    
- **ReplicaSets:** Ensuring a desired number of Pods are always running.
    
- **Deployments:** Managing the lifecycle, scaling, and updates of your application.
    
- **Services:** Providing stable network access to your application.
    

Understanding these core concepts is paramount for building and managing any application on Kubernetes. Next week, we'll explore more advanced topics!
