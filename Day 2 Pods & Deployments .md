# Difference between Container & Pod
A) Only difference is , in docker, Container goes with imperative approach by running necessary commands like docker run --name (etc). Whereas in Kubernetes, we follow declarative approach and those commands are wrtitten in YAML file, so that kubelet will make sure that container has the necessary volumes and networks to the container as mentioned in YAML file.(desired state). Pod is like a wrapper of kubernetes.

A pod can be single container or multiple containers (like sidecar or init etc.), so that both can have shared network or shared storage as required.

Pod.yaml can be stored in Github file and Devops engineers will write these files.

# Kubectl is command line tool for Kubernetes. 

Kubernetes = an orchestration system that manages many containers across nodes.

In Kubernetes, you don’t talk to containers directly. Instead:

You define YAML manifests (Pod, Deployment, Service, etc.).

The Kubernetes API Server interprets those.

The Kubelet on each node talks to Docker (or container runtime) to actually start/stop containers.

## Why kubectl Exists

Kubernetes isn’t a single engine like Docker. It’s a distributed system with multiple components:

API Server (central brain, exposes REST API)

etcd (database)

Scheduler (decides where Pods run)

Controller Manager (ensures desired state)

Kubelet (talks to container runtime on each node)

To “access Kubernetes directly,” you’d have to:

Send raw HTTPS REST calls to the API Server.

Example:

curl --header "Authorization: Bearer <TOKEN>" \
     https://<k8s-api-server>:6443/api/v1/namespaces/default/pods

This works (Kubernetes is entirely API-driven), but it’s very complex to use in practice.


kubectl is just a convenient CLI client for that API.

Instead of writing raw REST calls, you do:

kubectl get pods
kubectl apply -f deployment.yaml


Behind the scenes, kubectl converts this into REST requests to the API Server.

## Minikube Installation on EC2:

Minikube is just a local Kubernetes cluster designed for development and testing.

Since an EC2 instance is just a Linux VM, you can install Minikube there the same way you do on your laptop.

This lets you test manifests (pod.yaml, deployment.yaml, etc.) before moving to a real multi-node cluster.

# Requirements:

Instance type: at least t3.medium (2 vCPU, 4 GB RAM) → t2.micro is too small.

OS: Ubuntu/Debian/RedHat (Ubuntu 22.04 recommended).

Drivers: Since EC2 itself is a VM, you can’t use VirtualBox/VMware inside it. You need to use the --driver=none option (runs Minikube directly on the host without a nested VM).

Permissions: Root access required (use sudo).

# Installation Steps

On your EC2 Ubuntu instance:

# Update system
sudo apt-get update -y

# Install Docker
sudo apt-get install -y docker.io
sudo usermod -aG docker $USER
newgrp docker

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

Install kubectl on your EC2 instance

Run these commands after Minikube installation:

# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable
chmod +x kubectl

# Move to a directory in your PATH
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client

# Start Minikube without VM driver
minikube start --driver=none

# When you start Minikube with:

minikube start --driver=none

It automatically creates the kubeconfig file at:
~/.kube/config

Now you can run:

kubectl get nodes
kubectl get pods -A

#### Installation of Pod :

1. create a pod.yaml using vi
   vi pod.yml
2. Take the default pod.yml from kubernetes documentation for demo :
   
   apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80

3. Use the command kubectl create or apply as below :
   kubectl create -f pod.yml
4. To get pods & Pods details:
   kubectl get pods
   kubectl get pods -o wide
5. To SSH into minikube, use the below command , so that you will login to cluster :
   minikube ssh
6. To check the pod is running or not, copy the IP address of POD -o wide and run the below command :
   curl 172.18.0.3
7. You can delete pod , view logs, describe :
   kubectl delete pod nginx
   kubectl describe pod nginx
   kubectl logs nginx

### You can get all these kubectl commands from kubectl cheatsheet.

### Deployment.yml

# difference between container, pod & deployment :

Container : Think of a container as a single app running inside a box (e.g., one copy of Nginx web server).
Pod: If a container is one app, a Pod is a package/room that can hold one or a few tightly connected apps (like Nginx + sidecar container for logging).
Deployment : the manager that ensures you always have enough workers on duty, replaces them if they quit, and updates them smoothly.

# Deployment

A higher-level object that manages Pods.

Ensures the desired number of Pods are always running (self-healing, scaling, rolling updates).

If a Pod crashes, Deployment creates a new one automatically.
     
# A Deployment in Kubernetes is like a wrapper that manages ReplicaSets, and ReplicaSets manage Pods.
     
Deployment does not directly create Pods → it creates a ReplicaSet.

🔹 ReplicaSet

Ensures that the specified number of Pods are running at all times, like a controller.

If one Pod crashes, ReplicaSet will create a new Pod.

Deployment manages ReplicaSets → whenever you update the Deployment, it may create a new ReplicaSet (for the new version of Pods) and gradually scale down the old one.

🔹 Pod

The actual running unit on a Node.

Defined by the template inside the Deployment spec.

Pod runs containers (usually one main container, optionally sidecars). 

# Flow of Deployment.yml:

Deployment 
   ↓
Creates & manages ReplicaSet 
   ↓
ReplicaSet ensures Pods are running 
   ↓
Pods run containers

Before going to demo, delete the existing pods available in minikube using kubectl delete pod nginx

Now, check once for all the pods/deployments inside of cluster :
kubectl get all or kubectl get all -a

Create a deployment and paste the below yaml file :
vi deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

# kubectl apply -f deployment.yml

Check whether deployment is create or not using below command:
kubectl get deploy 
kubectl get rs(to check replica set controller created or not)
kubectl get pods( as deployment.yml will creats pods)

# Now delete pod and check whether replicaset is trying to create pod again to match the desired state. This is called Auto Healing .

kubectl delete pod nginx-name
# Open in another terminal -- kubectl get pods -w ( to see the what is happening to the pod while it is getting deleted).

# Now change the replicas to 3 in yaml file and apply once again, now check for the pods created . You can see 3 pods deployed.




