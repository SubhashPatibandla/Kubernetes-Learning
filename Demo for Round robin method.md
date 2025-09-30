# how Kubernetes does round-robin load balancing between pods, configured via a Service defined in service.yaml.

Here’s a complete minimal demo you can run locally (Minikube/kind) or on a cloud cluster.

1️⃣ Create a simple web app Deployment

We’ll run two replicas of a tiny web server that returns its pod name. That way you can see round-robin working.

deployment.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: web
        image: hashicorp/http-echo:0.2.3
        args:
          - "-text=Hello from $(POD_NAME)"
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
        ports:
        - containerPort: 5678


This runs hashicorp/http-echo, which echoes a message. We inject the pod name via env to see which pod responds.

2️⃣ Create a Service with round-robin load balancing

In Kubernetes, you don’t specify “round-robin” explicitly — ClusterIP or LoadBalancer Services already use round-robin across endpoints by default.

service.yaml:

apiVersion: v1
kind: Service
metadata:
  name: demo-service
spec:
  type: NodePort   # or NodePort if you’re on Minikube without LB
  selector:
    app: demo-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5678


By default, this Service will load balance requests across all pods selected by app: demo-app using round-robin.

3️⃣ Apply the manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml


Check:

kubectl get pods -o wide
kubectl get svc demo-service


On Minikube you can run:

minikube service demo-service


to open it in a browser, or use curl:

EXTERNAL_IP=$(minikube service demo-service --url)
curl $EXTERNAL_IP
curl $EXTERNAL_IP
curl $EXTERNAL_IP


You should see alternating responses:

Hello from demo-deployment-6db95d7b47-lc7gm
Hello from demo-deployment-6db95d7b47-bt4z9
Hello from demo-deployment-6db95d7b47-lc7gm
...


That’s Kubernetes’ built-in round-robin load balancing.

Notes

You don’t have to configure round-robin manually; Kubernetes Services + kube-proxy handle it automatically.

If you want sticky sessions (client always hits same pod), you’d add sessionAffinity: ClientIP in the Service spec.

On a local cluster without real load balancer, set type: NodePort and access via minikube service demo-service.
