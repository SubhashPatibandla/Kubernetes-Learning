### For each Deployment, we need Service to be attached.

# What is the need of Service.Yaml ?

We dont have service now and what are the issues we face ?

1. Lets say We have 3 Replica's for 100 concurrent users. Depends on no.of concurrent users, we will place the replica's, as a Devops engineer will take this decission.

   For some reason, one Pod gone down, because of Auto Healing Feature, Pod will come up. Thanks to Kubernetes (Replica Sets Controller).

   But the IP address going to change once the new Pod comes up.

   If you have already shared the older IP addresses to test/dev team, now everyone is in trouble as the POD IP got changed and the applciation is not reachable.

   Even in real world too, Google wont assign us to use IP address to reach out to google.

   Google will distribute the traffic by loadbalancer using the help of kubeproxy upon service.yaml and will load the traffic onto new IP address as required.

2. Even if i have the loadbalancer in between, if IP address changes then service also should face same problem right if it goes with IP address ?

   So, with service, we have another advantage called Service Discovery.

   In Service Discovery, it will find its service using Labels and Selectors.

   It will try to match the App name provided in Deployment.yaml with the app name provided in the Labels & Selectors.

   So that, we dont need to bother about IP addresses, if Labels and selectors are in Place.

check the Deployment.yml and Service.yml now to cross check the name of the App in labels and selectors

3. Service are classified into 3 types :

   a) Cluster IP
   b) Node Port
   c) Load Balancer(Expose to Outside world).

# Cluster IP :

User can communicate with PODS using this cluster IP Service Type , if user is inside of cluster.

# Node Port :

Lets say user is inside of your organaisation, but they dont have the access to Kubernetes cluster. 

But for Development/testing purpose, he/she wants to reach out to the Pod which is inside of K8s cluster.

Then we can change the Pod service type to Node Port & will assign a port in between the range 32000 - 370000(default range in K8s for Node port).

Now, user can able to access the pod using the NODE IP : NODE PORT( ex: 192.168.0.1:32007)

Note: (User should be aware of Worker Node IP Address, not the POD IP address).

# Load Balancer :

Lets say you want to expose your frontend application to external world.

Now you can update the service type to Loadbalancer. Then that POD will be assigned with external IP Address which can be mapped to Domain Name in DNS .

Lets say, in AWS , you are using EKS, if you change the svc type of Loadbalancer. Now with the help of Cloud COntroller Manager, AWS will 
assign the Elastic Load Balancer(Public IP addresses--static one) to your Kubernetes Cluster.

Ex: Lets say you have created Amazon.com as Loadbalancer, anyone can able to access your application.
But someone in your organisation wants to access, who already has access to your VPC and nodes, then you can assign service as Nodeport.
Now someone who has access to inside of your cluster, then in default, it will be provided with ClusterID service type.

## DEMO ##

Lets start the Minikube.

1. kuebctl get all ( to get all the details available in cluster)

2. lets clone the Docker application written in Python Django Language by Abhishek Veeramalla for this demo purpose.

   Git Clone https://  .git

3. CHange the directory to Python Web App using cd commands

4. Build the Docker image & push it to repository(before to that , login to your dockerhub using docker login command)

   docker build -t subhash058/python-simple-app-demo:v1

5. create Deployment.yml and update the image name & app name & container port as mentioned in docker file :

   apiVersion: apps/v1
   
kind: Deployment

metadata:

  name: sample-python-app
  
  labels:
  
    app: sample-python-app
    
spec:

  replicas: 2
  
  selector:
  
    matchLabels:
    
      app: sample-python-app
      
  template:
  
    metadata:
    
      labels:
      
        app: sample-python-app
        
    spec:
    
      containers:
      
      - name: python-app
      
        image: subhash058/python-simple-app-demo:v1
        
        ports:
        
        - containerPort: 8000

   
6. kubectl apply -f deployment.yml

7. kubectl get deploy -- to check deployment created or not

8. kubectl get pods -- to check the pods creared or not depends on deployment.yml

9. kubectl get pods -o wide -- to get the IP addresses of those pods.

10. kubectl delete pod PODNAME -- to delete one of the pods

Now new pods get created with new IP addresss, thanks to replica set.

11. Now service discovery concept comes into picture where we utilise Labels and selectors to identify the service instead of IP address.

# 12. Now connect to the cluster using minikube SSH.
    
  minikube SSH --command

13. lets try--  curl -L http://172.17.0.5:8000/demo ( you need to add demo as well as context route of application as that is what described in path url while writing application).

14. Now we can try to access the application as we're inside the cluster.

15. But we need someone from the organisation or outside of the organisation to access the application using Nodeport IP or Loadbalancer

# 15. Now, lets jump into Nodeport concept :

16. create of service.yaml

vi service.yml

apiVersion: v1

kind: Service

metadata:

  name: my-service (# any random name)
  
spec:

  type: NodePort
  
  selector:
  
    app: sample-python-app
    
  ports:
  
    - port: 80
    
      # By default and for convenience, the `targetPort` is set to
      
      # the same value as the `port` field.
      
      targetPort: 8000
      
      # Optional field
      
      # By default and for convenience, the Kubernetes control plane
      
      # will allocate a port from a range (default: 30000-32767)
      
      nodePort: 30007


17. Now lets apply --- kubectl apply -f service.yml

18. kuebctl get svc --- to get all the services created.

19. kubectl get svc -v=7 ( how the traffic is going inside cluster for the demo purpose)

    Now you could see the application has been mapped with Nodeport as type and POD IP address .

20. To get the minikube ip address use the below command :

    minikube ip -- you will get the node ip address.

21. now access the POD using the NODEIP:NODEPORT

    curl -L http://192.168.0.1:30007/demo

  Note: If you are doing this on your local server minikube version, you can access the application on your local browser too, as both are in same network.

  But if you're on EC2 instance, you need to allow the Nodeport in your security group as shown below :

  AWS EC2 instances sit behind a security group (firewall).

By default, only ports like 22 (SSH) and maybe 80/443 are open.

NodePorts (e.g., 30080, 32000) will be blocked unless you allow them.

Update the EC2 Security Group → allow inbound traffic on 30080 (or whichever NodePort you picked).


22. You can edit the service type in service.yaml and update the type of LoadBalancer:

    kubectl edit svc my-service

23. What Happens in Minikube (including on EC2)

Minikube runs Kubernetes locally (or inside your VM/EC2).

It doesn’t have a cloud provider (AWS/GCP/Azure) integration by default.

So when you create a LoadBalancer service in Minikube:

kubectl get svc --will show EXTERNAL-IP = <pending>

Because there is no cloud controller to actually allocate a load balancer.

Minikube provides a helper command:

minikube tunnel
    
With minikube tunnel, you might get an internal IP (like 10.x.x.x), but it won’t be a real public IP from AWS.

even this IP address cannot able to accessible, as this is just simulated by minikube tunnel from your internal EC2 machine. 

On EC2, that "external IP" is inside the EC2 instance’s network, not automatically a public internet IP.
   

Check the "Demo for Round Robin Method if interested"


   
