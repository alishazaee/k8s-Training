# Roles
The Role `alishazaee-role` defines permissions for a user or group to perform various operations on pods and services within the cluster. The specified verbs include **"get", "patch", "list", "create", "update", and "delete"**. The resources that these actions can be performed on are pods and services.

The RoleBinding `alishazaee-role-binding` binds the Role `alishazaee-role` to a User named `alishazaee`. This means that the user "alishazaee" will have the permissions defined in the Role "alishazaee-role" for interacting with pods and services in the cluster.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: alishazaee-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "patch" , "list", "create", "update", "delete"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alishazaee-role-binding
subjects:
- kind: User
  name: alishazaee
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: alishazaee-role
  apiGroup: rbac.authorization.k8s.io
```
now as we expected, we can get pods in the default namespace but we are not able to get the pods from the other namespaces.
```
ali@ali-shazaei ~/temp $ kubectl get pod
No resources found in default namespace.
ali@ali-shazaei ~/temp $ kubectl get pod -n microservice
Error from server (Forbidden): pods is forbidden: User "alishazaee" cannot list resource "pods" in API group "" in the namespace "microservice"
```