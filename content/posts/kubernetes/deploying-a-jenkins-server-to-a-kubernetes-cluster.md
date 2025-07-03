+++
title = 'Deploying a Jenkins Server to a Kubernetes Cluster'
date = 2025-07-03T19:13:52+01:00
draft = true
tags = ['devops', 'kubernetes']
+++

> _Part 2 of the **10 Kubernetes Projects in 10 Weeks** challenge_

Welcome to the second project in my Kubernetes challenge! This time, we're deploying **Jenkins**—a powerful CI/CD automation server—on a **Minikube Kubernetes cluster**, and we’ll focus on one crucial aspect: **persistent storage**.

Stateful apps like Jenkins need to store data across pod restarts—like pipeline configurations, plugins, and credentials. We'll achieve that using **Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** in Kubernetes.

Let’s dive in! 🏊‍♂️

---

## 🧠 What You'll Learn

- How to deploy Jenkins in a Kubernetes cluster using Minikube
- How to configure Persistent Volumes and Persistent Volume Claims
- Why persistence is essential for stateful applications like Jenkins

---

## 📦 Prerequisites

Before you start, make sure you have the following installed:

- ✅ [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- ✅ [kubectl](https://kubernetes.io/docs/tasks/tools/)
- ✅ [Helm](https://helm.sh/docs/intro/install/) (optional, but makes things easier)
- ✅ Basic familiarity with Kubernetes concepts (Pods, Services, PVCs)

---


## Why Persistent Storage for Jenkins?

Jenkins is a continuous integration/continuous delivery (CI/CD) automation server. It stores a lot of critical data:

- **Build history:** Logs, artifacts, and results of past builds.
    
- **Configuration:** Job definitions, plugin settings, user credentials.
    
- **Plugins:** Installed and configured plugins.
    
- **Workspaces:** Temporary data used during builds.
    

Without persistent storage, if a Jenkins pod restarts (due to an update, node failure, or scaling event), all this data would be lost, effectively resetting your Jenkins instance. This is where PVs and PVCs come into play.


## Understanding Persistent Volumes (PVs)

Think of a **Persistent Volume (PV)** as a piece of storage in your cluster that has been provisioned by an administrator (or dynamically by a storage provisioner). It's a cluster-wide resource, independent of any specific pod.

A PV abstracts the underlying storage infrastructure. It could be:

- NFS (Network File System) share
    
- iSCSI
    
- Cloud-specific storage (AWS EBS, Google Persistent Disk, Azure Disk)
    
- Local storage (as we'll use in Minikube for demonstration)
    

Key characteristics of a PV include:

- **Capacity:** How much storage it provides (e.g., 5Gi).
    
- **Access Modes:** How the storage can be mounted (e.g., `ReadWriteOnce` for single-node access, `ReadOnlyMany` for multiple read-only consumers, `ReadWriteMany` for multiple read-write consumers).
    
- **Reclaim Policy:** What happens to the volume when the PVC using it is deleted (e.g., `Retain`, `Recycle`, `Delete`).
    

## Understanding Persistent Volume Claims (PVCs)

While a PV is the actual storage resource, a **Persistent Volume Claim (PVC)** is a request for storage by a user or an application. It's like a consumer requesting a specific type and amount of storage.

A PVC specifies:

- **Amount of storage:** How much space is needed.
    
- **Access modes:** The desired way to access the storage.
    
- **Storage Class (optional but recommended):** A way to describe "classes" of storage (e.g., "fast-ssd", "backup-hdd").
    

Kubernetes then tries to find a suitable PV that matches the PVC's requirements. Once a PV is found and bound to a PVC, that PV is exclusively reserved for that PVC.

**Analogy:** A PV is like an empty apartment building (the physical storage). A PVC is like a tenant requesting an apartment with specific features (size, number of rooms). Kubernetes then assigns an available apartment (PV) that matches the tenant's request (PVC).


## Minikube Setup (Briefly)

For this project, we'll use Minikube, a tool that runs a single-node Kubernetes cluster locally. Minikube makes it easy to experiment with Kubernetes features without needing a full-blown cluster. It also supports local persistent volumes, which is perfect for our demonstration.

Ensure your Minikube cluster is running: `minikube start`

## Deploying Jenkins with PV/PVC

Let's define our Kubernetes resources. We'll create three YAML files: one for the PV, one for the PVC, and one for the Jenkins Deployment and Service.

### 1. Persistent Volume (PV) Definition (`jenkins-pv.yaml`)

This PV will use Minikube's host path, meaning it will store data directly on the Minikube VM's filesystem. This is suitable for local testing but not for production.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  labels:
    type: local
spec:
  storageClassName: manual # We'll manually bind this PV
  capacity:
    storage: 5Gi # 5 Gigabytes of storage
  accessModes:
    - ReadWriteOnce # Can be mounted as read-write by a single node
  hostPath:
    path: "/mnt/data/jenkins-volume" # Path inside the Minikube VM
  persistentVolumeReclaimPolicy: Retain # Keep the data even if PVC is deletedu
```

**Explanation:**

- `storageClassName: manual`: We're explicitly defining a storage class name. This allows us to bind a PVC to this specific PV.
    
- `capacity: storage: 5Gi`: We are provisioning 5 Gigabytes of storage.
    
- `accessModes: ReadWriteOnce`: This PV can be mounted by one node in read-write mode.
    
- `hostPath: path: "/mnt/data/jenkins-volume"`: This tells Minikube to create a directory at this path inside its VM to store the data.
    
- `persistentVolumeReclaimPolicy: Retain`: If the PVC using this PV is deleted, the PV and its data will remain. This is a safe choice for important data.

### 2. Persistent Volume Claim (PVC) Definition (`jenkins-pvc.yaml`)

This PVC will request 5Gi of storage with `ReadWriteOnce` access, matching our PV.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
spec:
  storageClassName: manual # Must match the PV's storageClassName
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi # Requesting 5 Gigabytes
```


**Explanation:**

- `storageClassName: manual`: This PVC will look for a PV with the `manual` storage class.
    
- `accessModes: ReadWriteOnce`: It requests the ability to be mounted read-write by a single node.
    
- `resources: requests: storage: 5Gi`: It requests 5 Gigabytes of storage.
    
### 3. Jenkins Deployment and Service (`jenkins-deployment.yaml`)

This defines our Jenkins deployment, which will use the PVC, and a Service to expose Jenkins.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
  labels:
    app: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts # Using the Long Term Support Jenkins image
          ports:
            - name: http-port
              containerPort: 8080
            - name: jnlp-port
              containerPort: 50000
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home # Jenkins stores its data here
          livenessProbe: # Health check to ensure Jenkins is running
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 5
          readinessProbe: # Readiness check before sending traffic
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 5
      volumes:
        - name: jenkins-home
          persistentVolumeClaim:
            claimName: jenkins-pvc # Referencing our PVC
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
spec:
  selector:
    app: jenkins
  type: NodePort # Expose Jenkins via a NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30000 # Access Jenkins on your host at port 30000
      name: http
    - port: 50000
      targetPort: 50000
      name: jnlp # For Jenkins agents
```

**Explanation:**

- **Deployment:**
    
    - `image: jenkins/jenkins:lts`: Specifies the Jenkins Docker image.
        
    - `volumeMounts`: This section mounts the `jenkins-home` volume at `/var/jenkins_home` inside the container, which is where Jenkins expects its data to be.
        
    - `volumes`: This section defines the `jenkins-home` volume and links it to our `jenkins-pvc` using `persistentVolumeClaim: claimName: jenkins-pvc`.
        
    - `livenessProbe` and `readinessProbe`: These ensure Kubernetes knows when Jenkins is healthy and ready to receive traffic.
        
- **Service:**
    
    - `type: NodePort`: Exposes the Jenkins service on a static port on each node's IP. Minikube will make this accessible from your host machine.
        
    - `nodePort: 30000`: We've chosen `30000` for easy access.
        
## Steps to Deploy

1. **Save the YAML files:** Save the content above into `jenkins-pv.yaml`, `jenkins-pvc.yaml`, and `jenkins-deployment.yaml` respectively.
    
2. **Apply the PV:**
    
    ```
    kubectl apply -f jenkins-pv.yaml
    ```
    
    Verify it's created and available:
    
    ```
    kubectl get pv
    ```
    
    You should see `jenkins-pv` with status `Available`.
    
3. **Apply the PVC:**
    
    ```
    kubectl apply -f jenkins-pvc.yaml
    ```
    
    Verify it's created and bound:
    
    ```
    kubectl get pvc
    ```
    
    You should see `jenkins-pvc` with status `Bound` to `jenkins-pv`.
    
4. **Apply the Jenkins Deployment and Service:**
    
    ```
    kubectl apply -f jenkins-deployment.yaml
    ```
    
    Monitor the deployment:
    
    ```
    kubectl get pods -l app=jenkins -w
    ```
    
    Wait until the Jenkins pod is `Running` and `Ready`. This might take a few minutes as Jenkins downloads plugins and initializes.
    

## Verification

Once the Jenkins pod is running, you can access it:

1. **Get the Minikube IP:**
    
    ```
    minikube ip
    ```
    
2. **Access Jenkins:** Open your web browser and navigate to `http://<minikube-ip>:30000`.
    

You should see the Jenkins setup wizard. Go through the initial setup (you can find the initial admin password by checking the pod logs: `kubectl logs <jenkins-pod-name>`).

**To test persistence:**

1. Complete the Jenkins setup, create a sample job, or install a plugin.
    
2. Delete the Jenkins pod:
    
    ```
    kubectl delete pod -l app=jenkins
    ```
    
    Kubernetes will automatically create a new pod for the deployment.
    
3. Once the new pod is running, refresh your browser. You should find your Jenkins instance exactly as you left it, with all your configurations, jobs, or installed plugins intact. This confirms that the persistent storage is working!
    

## Conclusion

This week, we successfully deployed a Jenkins server on Minikube, demonstrating the critical role of Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) in managing stateful applications in Kubernetes. You've learned:

- Why persistent storage is essential for applications like Jenkins.
    
- The distinction between a cluster-wide PV (the actual storage) and a user-requested PVC (the claim for storage).
    
- How to define and link PVs and PVCs in your Kubernetes manifests.
    
- How to configure a Deployment to use a PVC for its data.
    

This foundational understanding of persistent storage is vital as you continue your Kubernetes journey. Next week, we'll explore another exciting aspect of Kubernetes!
