## How to Write YAML File ## 

# 1. What is YAML?

YAML → “YAML Ain’t Markup Language”

A human-friendly data format used for configuration files.

Preferred in DevOps & Kubernetes because:

Easy to read/write

Handles complex configurations

Supported by most DevOps tools

👉 Think of it like a settings file written in plain English style.

<img width="817" height="491" alt="image" src="https://github.com/user-attachments/assets/bdb0eeba-ac09-42ea-a537-4ca1670f58e0" />

# 2. YAML vs JSON
Feature	      JSON	          YAML
Syntax	      Uses { }, [ ], "	Uses indentation & colons
Readability	    Harder (lots of braces & quotes)	Easier (clean & minimal)
Comments	❌ Not supported	✅ Supported using #
Use case	Data exchange (APIs)	Configurations (K8s, CI/CD)

👉 Same data in JSON vs YAML:

JSON

{
  "name": "Subhash",
  "skills": ["Docker", "Kubernetes", "Terraform"]
}


YAML

name: Subhash
skills:
  - Docker
  - Kubernetes
  - Terraform

# 3. YAML Use Cases in a DevOps Engineer Role

Kubernetes manifests → Pods, Deployments, Services, ConfigMaps, Secrets.

CI/CD pipelines → GitHub Actions, GitLab CI.

Infrastructure as Code → Ansible playbooks, Helm charts, Terraform (partially).

Monitoring tools → Prometheus, Grafana configs.

Cloud configurations → AWS CloudFormation (YAML supported).

👉 In short: A DevOps engineer writes YAML almost every day.

# 4. Key Concepts in YAML

a) Key-Value pairs

name: Subhash


b) Lists

fruits:
  - apple
  - banana


c) Nested Objects

address:           # <-- nested object
  city: Hyderabad
  country: India


d) Lists of Objects

employees:        # <-- list of objects
  - name: Alice
    role: Developer
  - name: Bob
    role: Tester


e) Comments

# This is a comment
app: nginx

Example : 
apiVersion: v1          # Kubernetes API version for Pod
kind: Pod               # Resource type: Pod
metadata:               # Metadata = identifying information
  name: nginx-pod       # Pod name (must be unique in namespace)
  labels:               # Labels help identify and group objects
    app: nginx
spec:                   # Specification of the pod
  containers:           # List of containers in this pod
    - name: nginx       # Container name
      image: nginx:latest # Docker image to use
      ports:            # List of ports exposed by this container
        - containerPort: 80  # Port on which container listens
