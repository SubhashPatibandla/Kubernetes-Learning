### Deployment.yml

# Difference between container, pod & deployment :

Container : Think of a container as a single app running inside a box (e.g., one copy of Nginx web server).

Pod: If a container is one app, a Pod is a package/room that can hold one or a few tightly connected apps (like Nginx + sidecar container for logging).

Deployment : the manager that ensures you always have enough workers on duty, replaces them if they quit, and updates them smoothly.

# Deployment

A higher-level object that manages Pods.

Ensures the desired number of Pods are always running (self-healing, scaling, rolling updates).

If a Pod crashes, Deployment creates a new one automatically.
     
A Deployment in Kubernetes is like a wrapper that manages ReplicaSets, and ReplicaSets manage Pods.
     
Deployment does not directly create Pods → it creates a ReplicaSet.

# ReplicaSet

Ensures that the specified number of Pods are running at all times, like a controller.

If one Pod crashes, ReplicaSet will create a new Pod.

Deployment manages ReplicaSets → whenever you update the Deployment, it may create a new ReplicaSet (for the new version of Pods) and gradually scale down the old one.

# Pod

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

Create a deployment(vi deployment.yml) and paste the below yaml file :


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

Now delete pod and check whether replicaset is trying to create pod again to match the desired state. This is called Auto Healing .

kubectl delete pod nginx-name

Open in another terminal -- kubectl get pods -w ( to see the what is happening to the pod while it is getting deleted).

Now change the replicas to 3 in yaml file and apply once again, now check for the pods created . You can see 3 pods deployed.



