# Role Base Access Control (RBAC) on Kubernetes

To demonstrate the working of RBAC, following scenario is considered. 

## Role and RoleBinding - Scenario 1
You are the administrator of a Kubernetes cluster that runs a mission-critical application. You want to ensure that only authorized users are able to deploy new pods and services to the cluster. You have been asked to implement Role-Based Access Control (RBAC) to achieve this.

Your task is to:
1. Create a new Role named `pod-deployer` that grants permissions to `create` and `delete` pods in the `default` namespace.
2. Create a new RoleBinding named `pod-deployer-binding` that binds the `pod-deployer` Role to the `asad-hanif` user.
3. Verify that the `asad-hanif` user is able to create and delete pods in the `default` namespace, but not in other namespaces.

## Solution 

Follwing are the steps of solution:

1. Creating a new Role named **pod-deployer**.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-deployer
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "create", "delete"]
```

2. Creating a new RoleBinding named **pod-deployer-binding**.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-deployer-binding
  namespace: default
subjects:
- kind: User
  name: asad-hanif
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role 
  name: pod-deployer
  apiGroup: rbac.authorization.k8s.io
```

With this configuration, the user `asad-hanif` will be able to `create` and `delete` Pods in the `default` namespace, but will not have permissions to perform any other actions on resources in that namespace.

Now create Role and RoleBinding using following command:

`kubectl apply -f role-and-rolebinding-scenario-1.yaml`

3. Verify that the user `asad-hanif` can create and delete pods in the `default` namespace:

```
kubectl auth can-i create pods --namespace=default --as=asad-hanif

yes
```

```
kubectl auth can-i delete pods --namespace=default --as=asad-hanif

yes
```

```
kubectl auth can-i create pods --namespace=other --as=asad-hanif

no
```

```
kubectl auth can-i delete pods --namespace=other --as=asad-hanif

no
```

## ClusterRole and ClusterRoleBinding - Scenario 2
Suppose you have a Kubernetes cluster with **multiple namespaces** and **several teams** working on different applications. You want to create a set of roles and role bindings that ensure that each team has access only to the resources they need in their respective namespaces.

- The **dev** team should be able to create, update, and delete deployments, pods, and services in the **dev** namespace.
- The **qa** team should be able to create, update, and delete deployments, pods, and services in the **qa** namespace.
- The **prod** team should be able to create, update, and delete deployments, pods, and services in the **prod** namespace, but only if the labels on the resources match a predefined set of labels.
- The **admin** team should be able to create, update, and delete deployments, pods, services, and namespaces across the entire cluster.
- The **viewer** role should be able to view deployments, pods, and services in all namespaces, but not modify them.

Create appropriate roles and role bindings to achieve the above access control requirements.

## Solution 

Follwing are the steps of solution:


0. Create all namespaces.

```
kubectl create namespace dev
kubectl create namespace qa
kubectl create namespace prod
```

1. The **dev** team should be able to create, update, and delete deployments, pods, and services in the **dev** namespace.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: dev-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "pods", "services"]
  verbs: ["create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-role-binding
roleRef:
  kind: ClusterRole
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: dev
  apiGroup: rbac.authorization.k8s.io
```

2. The **qa** team should be able to create, update, and delete deployments, pods, and services in the **qa** namespace.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: qa-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "pods", "services"]
  verbs: ["create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: qa-role-binding
roleRef:
  kind: ClusterRole
  name: qa-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: qa
  apiGroup: rbac.authorization.k8s.io
```

3. The **prod** team should be able to create, update, and delete deployments, pods, and services in the **prod** namespace, but only if the labels on the resources match a predefined set of labels.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prod-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "pods", "services"]
  verbs: ["create", "update", "delete"]
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "watch", "list"]
  resourceNames:
  - frontend
  - backend
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prod-role-binding
roleRef:
  kind: ClusterRole
  name: prod-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: prod
  apiGroup: rbac.authorization.k8s.io
```

4. The **admin** team should be able to create, update, and delete deployments, pods, services, and namespaces across the entire cluster.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: admin-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "pods", "services", "namespaces"]
  verbs: ["create", "update", "delete", "get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: admin-role-binding
roleRef:
  kind: ClusterRole
  name: admin-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

5. The **viewer** role should be able to view deployments, pods, and services in all namespaces, but not modify them.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: viewer-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "pods", "services"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: viewer-binding
subjects:
- kind: Group
  name: viewer
roleRef:
  kind: ClusterRole
  name: viewer-role
  apiGroup: rbac.authorization.k8s.io
```

