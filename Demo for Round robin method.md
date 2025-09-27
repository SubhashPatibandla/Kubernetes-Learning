Kubernetes Demo Manifest

# deployment-service-demo.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: demo-app
  
spec:

  replicas: 3
  
  selector:
  
    matchLabels:
    
      app: demo-app
      
  template:
  
    metadata:
    
      labels:
      
        app: demo-app
        
    spec:
    
      containers:
      
      - name: demo-container
      
        image: hashicorp/http-echo:0.2.3
        
        args:
        
          - "-text=$(POD_NAME)"
          
        env:
        
          - name: POD_NAME
          
            valueFrom:
            
              fieldRef:
              
                fieldPath: metadata.name
                
        ports:
        
        - containerPort: 5678
        
---
apiVersion: v1

kind: Service

metadata:

  name: demo-service
  
spec:

  selector:
  
    app: demo-app
    
  ports:
  
    - port: 80
    
      targetPort: 5678
      
  type: ClusterIP
  

How to Run the Demo

1️⃣ Apply the manifest

kubectl apply -f deployment-service-demo.yaml


2️⃣ Verify pods are running

kubectl get pods


You should see something like:

NAME                        READY   STATUS    RESTARTS   AGE

demo-app-7f4b9d8c7c-abcde  1/1     Running   0          1m

demo-app-7f4b9d8c7c-fghij  1/1     Running   0          1m

demo-app-7f4b9d8c7c-klmno  1/1     Running   0          1m

3️⃣ Run a curl loop from inside the cluster

kubectl run curlpod --image=radial/busyboxplus:curl -i --tty --rm -- /bin/sh


Inside the pod:

while true; do curl demo-service; sleep 1; done


You should see output like:

demo-app-7f4b9d8c7c-abcde

demo-app-7f4b9d8c7c-fghij

demo-app-7f4b9d8c7c-klmno

demo-app-7f4b9d8c7c-abcde

demo-app-7f4b9d8c7c-fghij

demo-app-7f4b9d8c7c-klmno


✅ This clearly shows round-robin load balancing between pods.


4️⃣ Optional — Check logs of each pod

kubectl logs demo-app-7f4b9d8c7c-abcde

kubectl logs demo-app-7f4b9d8c7c-fghij

kubectl logs demo-app-7f4b9d8c7c-klmno


This will confirm which pod served which request.
