What is Ingress ?

Before to that, we have discussed about Services .

Services is offering Round Robin Load Balancing method and Exposing the application to Outside world as well using LoadBalancer Service Type.

So people were using Kubernetes with Service concept until Dec 2015. Then, everyone started complaining about LoadBalancer Mode of Service. 

1) Actually they have earlier used legacy Load Balancers like Nginx, F5 etc(enterprise level load balancers with feature like ratio based , sticky sessions, path based, domain based

white listing, black listing etc.)

Once they migrated to Kubernetes, they are happy with Autohealing, Autoscaling, Service Discovery using Services etc.

But the Load Balancing mechanism provided by Services is just Round Robin method(very simple load balancing method). They're missing the enterprise level features which were mentioned earlier.

2) Using Load Balancer mode Service for the frontend application, you're exposing the app to Outside world. But the bigger applications like amazon had 1000's of microservices and 100's of them

will be on frontend which needs to be exposed on outside world. Then they need to configure 100's of load balancers for each and every application separately.

Because of that, maintenance cost to carry these many static IP addresses for Load balanacers became heavy.

## Solution is Ingress :

So Kubernetes understood this problem being faced by organisations who are developing those applications. They came up with a solution as below : 

Usually if you write deployment.yaml, kubelet resource will help you to create pods in worker node.

and if your write service.yaml, kubeproxy resource will help you to create NodePort or LoadBalancer Service type.

So here we're going to write Ingress.yaml and we're asking to deploy Ingress controller from the Organisation providing the Load Balancer.

1. Create a Ingress (ingress.yaml ) with Path based or Host based as you need.

2. In market, as we have many enterprise level Load Balancers, you can ask them to provide Ingress controller and deploy it as a Pod in your cluster & expose it to outside world using LB.

3. Now, lets say Nginx Ingress Controller will create a Load Balancer on Pod. So whenever the traffic hits the domain named Amazon.com(example), then the Load balancer will take the traffic

4. Will forward this traffic to Ingress Controller. Ingress Controller always reads the Ingress resource and will forward the traffic to the necessary applciation as required.

✅ So to summarize:

LoadBalancer Service → points to Ingress Controller Pods.

Ingress Controller → reads Ingress YAML rules.

Ingress YAML → tells controller where to route traffic inside the cluster.

Its a one time activity to decide which Ingress COntroller type of Laod Balancer they want.

Using One Ingress LB, you can serve 100's of applications now.

## DEMO :

🚀 Demo: Nginx Ingress with Path-Based Routing on Minikube

Step 1️⃣ – Start Minikube

minikube start --driver=docker

Step 2️⃣ – Enable Ingress Addon (Nginx Ingress Controller)

minikube addons enable ingress


👉 This installs Nginx Ingress Controller inside your Minikube cluster.

It also creates a Service of type NodePort (not LoadBalancer, since Minikube isn’t on cloud).

Step 3️⃣ – Deploy Two Sample Apps

Let’s deploy two apps (app1 and app2) to demonstrate path-based routing.

# cart-app.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cart-app
  template:
    metadata:
      labels:
        app: cart-app
    spec:
      containers:
      - name: cart-app
        image: hashicorp/http-echo
        args:
          - "-text=Hello from Cart Service"
---
apiVersion: v1
kind: Service
metadata:
  name: cart-app
spec:
  selector:
    app: cart-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678


# app2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: payment-app
  template:
    metadata:
      labels:
        app: payment-app
    spec:
      containers:
      - name: payment-app
        image: hashicorp/http-echo
        args:
          - "-text=Hello from Payment Service"
---
apiVersion: v1
kind: Service
metadata:
  name: payment-app
spec:
  selector:
    app: payment-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678



Apply them:

kubectl apply -f app1.yaml

kubectl apply -f app2.yaml

Step 4️⃣ – Create Ingress Resource (Path-Based Routing)

Now, we tell Kubernetes (via Ingress YAML) how to route requests.

#ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shop-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /cart
        pathType: Prefix
        backend:
          service:
            name: cart-app
            port:
              number: 80
      - path: /payment
        pathType: Prefix
        backend:
          service:
            name: payment-app
            port:
              number: 80



Apply it:

kubectl apply -f ingress.yaml

Step 5️⃣ – Get Minikube IP

Since Minikube doesn’t give a cloud LoadBalancer, we access via Minikube Node IP.

minikube ip


Example output:

192.168.49.2

Step 6️⃣ – Test the Routing

Now use curl or browser:

curl http://192.168.49.2/app1

# Output: Hello from App1

curl http://192.168.49.2/app2

# Output: Hello from App2


✅ This proves that Ingress Controller (Nginx) is taking traffic from 192.168.49.2 and routing based on path rules to the correct backend Service.


