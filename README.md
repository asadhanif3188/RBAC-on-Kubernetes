# Role Base Access Control (RBAC) on Kubernetes

To demonstrate the working of RBAC, following scenario is considered. 

## Role and RoleBinding - Scenario
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

