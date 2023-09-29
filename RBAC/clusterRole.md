# ClusterRole
A ClusterRole is a Kubernetes object that defines a set of permissions within a cluster. It specifies what actions (verbs) can be performed on which resources (API groups, resources, and subresources) by whom (subjects). 

In this scenario, the ClusterRole named `alishazaee-admin` grants permissions to perform "get", "watch", and "list" operations on pods within the cluster. This means that any user or group that is assigned this ClusterRole will have the ability to perform these actions on pods.

To connect the ClusterRole to a user, a ClusterRoleBinding is created. A ClusterRoleBinding is another Kubernetes object that binds a ClusterRole to a user or group. In this configuration, a ClusterRoleBinding named "alishazaee-clusterrolebinding" is created to bind the ClusterRole "alishazaee-admin" to a User named "alishazaee". This means that the user "alishazaee" will have the permissions defined in the ClusterRole "alishazaee-admin" for interacting with pods in the cluster.

```
ali@ali-shazaei ~/temp $ cat clusterRole.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: alishazaee-admin
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: alishazaee-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: alishazaee-admin
subjects:
- kind: User
  name: alishazaee 
  apiGroup: rbac.authorization.k8s.io
```