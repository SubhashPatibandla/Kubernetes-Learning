# Kubernetes-Learning

## Why Kubernetes?

Docker alone is mainly for containerization — it packages and runs a single container (or a few, manually). But in real-world production you need more:

Docker = container engine; Kubernetes = container orchestration platform.
They’re complementary, not mutually exclusive. In fact Kubernetes uses Docker/containerd as a runtime under the hood (though containerd is more common now).

| Feature                   | Docker | Kubernetes |
|---------------------------|--------|-------------|
| Containerization          | ✅      | Uses container runtime (Docker/containerd) |
| Multi-host Orchestration  | ❌ (manual) | Orchestrates containers across multiple machines |
| Auto-healing              | ❌      | Self-healing — restarts pods, re-schedules on other nodes |
| Load Balancing            | Manual | Built-in service abstraction & load balancing |
| Rolling Updates           | Manual | Declarative deployments with automatic rollback |
| Config/Secrets mgmt       | Manual | Built-in secrets/config maps |
| Ecosystem/Cloud support   | Limited | Strong |
| Scaling   | Manual | Auto-scaling pods based on CPU/memory |



## Kubernetes Architecture :

                +---------------------------+
                |     Control Plane          |
                |---------------------------|
 kubectl  --->  | API Server                 |
                | etcd (state store)         |
                | Controller Manager         |
                | Scheduler                  |
                +---------------------------+
                           |
       ----------------------------------------------------
       |                     |                     |
+--------------+     +--------------+      +--------------+
| Worker Node1 |     | Worker Node2 |      | Worker Node3 |
|--------------|     |--------------|      |--------------|
| kubelet      |     | kubelet      |      | kubelet      |
| kube-proxy   |     | kube-proxy   |      | kube-proxy   |
| containerd   |     | containerd   |      | containerd   |
| Pods/Containers|   | Pods/Containers|    | Pods/Containers|
+--------------+     +--------------+      +--------------+

## Control Plane

| Component              | Role                                                                        |
| ---------------------- | --------------------------------------------------------------------------- |
| **API Server**         | Front-end of Kubernetes. All `kubectl` and internal components talk to it.  |
| **etcd**               | Key-value store for cluster state (declarative config).                     |
| **Controller Manager** | Ensures cluster state matches desired state (e.g., replicaset controllers). |
| **Scheduler**          | Decides on which node a new Pod runs, based on resources/affinity.          |

## Worker Node 

| Component             | Role                                                                  |
| --------------------- | --------------------------------------------------------------------- |
| **kubelet**           | Agent on each node that talks to API Server, starts/stops containers. |
| **Container Runtime** | e.g., containerd or Docker engine — actually runs the containers.     |
| **kube-proxy**        | Manages networking, load balancing, cluster IP routing.               |
| **Pods**              | Smallest deployable unit. Can have one or more containers.            |
