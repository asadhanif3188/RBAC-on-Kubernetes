# Role Base Access Control (RBAC) on Kubernetes

To demonstrate the working of RBAC, following scenario is considered. 

## Scenario
You are the administrator of a Kubernetes cluster that runs a mission-critical application. You want to ensure that only authorized users are able to deploy new pods and services to the cluster. You have been asked to implement Role-Based Access Control (RBAC) to achieve this.

Your task is to:
1. Create a new Role named `pod-deployer` that grants permissions to `create` and `delete` pods in the `default` namespace.
2. Create a new RoleBinding named `pod-deployer-binding` that binds the `pod-deployer` Role to the `asad-hanif` user.
3. Verify that the `asad-hanif` user is able to create and delete pods in the `default` namespace, but not in other namespaces.

## Solution 

