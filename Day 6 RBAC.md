## What is Kubernetes RBAC ?

Its very simple, but complicated. As its related to Security , we need to be very careful while implementing RBAC.

So Kubernetes Broadly classified into 2 types :

Users and Service Accounts.

If you remember the session IAM in AWS, then it will be lot easy.

So in RBAC, Roles will be assigned with Permissions. Users will be assigned with those roles using Role Binding concept.

Similarly , Service accounts on Pods will be assigned with Roles using Role Binding concept again.

## DEMO:

Alright 😄 — let’s go step-by-step for a full RBAC demo on Minikube, with two parts:

User-level RBAC (example: restricting a user to view-only access)

ServiceAccount-level RBAC (example: granting a service account permission to create pods).

We’ll include YAML files and commands with explanations for every line so it’s easy to follow and explain to others.

PART 1 — USER-LEVEL RBAC DEMO
Goal:

Create a user alice and allow her only read-only access to pods in the default namespace.

Step 1 — Create RBAC Role YAML (role-view-pods.yaml)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default  # 1. Limits this role to 'default' namespace
  name: pod-viewer     # 2. Role name
rules:
  - apiGroups: [""]      # 3. "" means core API group (pods are core resources)
    resources: ["pods"]  # 4. Specifies the resource type (pods)
    verbs: ["get", "list", "watch"]  # 5. Allowed actions: get/list/watch only


Explanation of each line:

namespace: Role is namespace-scoped; we set it to default so it applies only there.

name: Name of the role.

apiGroups: "" means the core API group (pods, services, etc.).

resources: The object(s) this role can act upon — here, "pods".

verbs: Actions allowed. "get", "list", "watch" are read-only.

Step 2 — Apply Role
kubectl apply -f role-view-pods.yaml

Step 3 — Create RoleBinding for User alice
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: view-pods-binding
  namespace: default       # Scope of binding
subjects:
  - kind: User            # 1. Subject type is User
    name: alice          # 2. User's name
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role              # 3. Reference to a Role
  name: pod-viewer
  apiGroup: rbac.authorization.k8s.io


Explanation:

kind: User → This binding applies to a user, not a group or service account.

name: User name — must match kubeconfig context user name.

roleRef: References the Role we created earlier.

Step 4 — Create kubeconfig context for alice

We simulate a new user by creating a separate kubeconfig entry.

kubectl config set-context alice-context \
  --cluster=minikube \
  --namespace=default \
  --user=alice


Explanation:
This creates a new context in kubeconfig for alice. It doesn’t create a real OS user — RBAC works based on this context user name.

Step 5 — Test as alice

Switch to alice-context:

kubectl config use-context alice-context


Test access:

kubectl get pods       # ✅ Allowed
kubectl delete pod nginx-pod  # ❌ Forbidden (no delete permission)


Key point: RBAC enforces what a user can do, not the user creation. The context must match the user name in RoleBinding.

PART 2 — SERVICE ACCOUNT LEVEL RBAC DEMO
Goal:

Create a service account that can create pods in default namespace.

Step 1 — Create Service Account
kubectl create serviceaccount pod-creator-sa --namespace=default


Explanation:
This creates a service account named pod-creator-sa in the default namespace.

Step 2 — Create Role YAML (role-create-pods.yaml)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-creator
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "get", "list", "watch"]


Explanation:
Similar to earlier role, but this time verbs include "create" so the service account can create pods.

Step 3 — Create RoleBinding YAML (rolebinding-pod-creator.yaml)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-creator-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: pod-creator-sa
    namespace: default
roleRef:
  kind: Role
  name: pod-creator
  apiGroup: rbac.authorization.k8s.io


Explanation:
Binds the pod-creator Role to pod-creator-sa service account.

Step 4 — Apply YAMLs
kubectl apply -f role-create-pods.yaml
kubectl apply -f rolebinding-pod-creator.yaml

Step 5 — Test Service Account RBAC

Get the service account token:

SECRET_NAME=$(kubectl get sa pod-creator-sa -n default -o jsonpath="{.secrets[0].name}")
TOKEN=$(kubectl get secret $SECRET_NAME -n default -o jsonpath="{.data.token}" | base64 --decode)


Create a kubeconfig for this SA:

kubectl config set-credentials pod-creator-sa --token=$TOKEN
kubectl config set-context pod-creator-context --cluster=minikube --namespace=default --user=pod-creator-sa
kubectl config use-context pod-creator-context


Test:

kubectl run testpod --image=nginx   # ✅ Allowed
kubectl delete pod testpod          # ❌ Forbidden (no delete permission)


✅ Summary of what we learned in demo:

User-level RBAC → controlled via Role + RoleBinding to a user in kubeconfig.

ServiceAccount-level RBAC → controlled via Role + RoleBinding to a ServiceAccount, with token-based authentication.

