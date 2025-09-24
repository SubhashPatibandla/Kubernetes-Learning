## What is a Manifest File?

A manifest file (YAML/JSON) tells Kubernetes what you want to run.

Kubernetes control plane components (API Server, Scheduler, Controller Manager, etcd) read it and make sure your worker nodes execute that plan.

Think of it as:

📜 Manifest = Declaration → 🧠 Control Plane = Brain → 💪 Worker Nodes = Muscles

### Common Manifest Files & Their Importance:

# 1. Pod Manifest (Pod.yaml)

Purpose: Smallest deployable unit; defines containers, images, ports.

Importance: Tells Kubernetes what containerized workload to run.

Architecture Link:

API Server accepts the request.

Scheduler decides which Node runs the Pod.

Kubelet runs the Pod on that Node.

etcd stores the Pod state.

# 2. Deployment Manifest (Deployment.yaml)

Purpose: Ensures desired number of replicas, rolling updates, rollback.

Importance: Provides scalability & self-healing for Pods.

Architecture Link:

Deployment Controller (Controller Manager) keeps checking if the number of Pods = desired replicas.

If one Pod fails, it creates another.

# 3. Service Manifest (Service.yaml)

Purpose: Exposes Pods to internal/external traffic.

Importance: Provides stable networking & load balancing for Pods.

Architecture Link:

kube-proxy (on worker nodes) updates iptables/IPVS rules.

DNS (CoreDNS) gives a consistent name for services.

# 4. ConfigMap & Secret Manifests (ConfigMap.yaml, Secret.yaml)

Purpose: Externalize configuration (non-sensitive in ConfigMap, sensitive in Secret).

Importance: Makes workloads flexible and secure without baking configs into images.

Architecture Link:

API Server stores them.

Kubelet injects them into Pods (as env vars or volumes).

# 5. Ingress Manifest (Ingress.yaml)

Purpose: Routes HTTP/HTTPS traffic into Services (layer 7 load balancing).

Importance: Provides user-friendly URLs, SSL termination, routing.

Architecture Link:

Needs Ingress Controller (like Nginx, Traefik, AWS ALB Controller) running in the cluster.

Works with API Server + kube-proxy + networking plugins.

# 6. PersistentVolume (PV) & PersistentVolumeClaim (PVC) Manifests

Purpose: Handle storage (databases, logs, app data).

Importance: Decouples storage from Pods (Pods are ephemeral).

Architecture Link:

Controller Manager (Persistent Volume Controller) binds PVC to PV.

Kubelet mounts storage into the Pod.

# 7. Namespace Manifest (Namespace.yaml)

Purpose: Logical isolation within a cluster (multi-team, multi-project).

Importance: Helps in organizing & securing resources.

Architecture Link:

API Server enforces namespace isolation.

RBAC policies applied at namespace level.

# 8. Role & RoleBinding / ClusterRole & ClusterRoleBinding

Purpose: Access control (RBAC).

Importance: Controls who can do what in the cluster.

Architecture Link:

API Server enforces RBAC policies.

## Summary 

### 📊 Kubernetes Manifest Files & Their Role

| **Manifest File**      | **Purpose**                          | **Connected Component**                    |
|-------------------------|--------------------------------------|--------------------------------------------|
| **Pod**                | Run container workload               | API Server, Scheduler, Kubelet, etcd       |
| **Deployment**         | Replica management & rollout         | Controller Manager                         |
| **Service**            | Stable networking & load balancing   | kube-proxy, CoreDNS                        |
| **ConfigMap / Secret** | Config mgmt & sensitive data         | API Server, Kubelet                        |
| **Ingress**            | HTTP/HTTPS routing                   | Ingress Controller, API Server             |
| **PV / PVC**           | Persistent storage                   | PV Controller, Kubelet                     |
| **Namespace**          | Resource isolation                   | API Server, RBAC                           |
| **RBAC**               | Security & permissions               | API Server                                 |


### Sample Folder Structure of Each Microservice :

<img width="537" height="327" alt="image" src="https://github.com/user-attachments/assets/a965f43e-1b06-42d1-9834-51fb9e29676c" />




