# User Authentication
each developer team works in a different namespace. we want to restrict access to only their namespaces so they don't accidentally break something in the namespace of the other team. we use RBAC or role based access control to do this.

In Kubernetes, the concepts of roles, role bindings, and users are interconnected to define access and permissions within the cluster. Here's an explanation of these concepts and their relationship:

1. Role:
   - A role is a Kubernetes object that defines a set of rules and permissions for accessing resources within a specific namespace.
   - Roles are created using the `Role` API object and defined in YAML manifests.
   - Roles can specify permissions for various resources, such as pods, services, secrets, config maps, etc.
   - Roles define what actions a user or service account can perform on the specified resources, like create, read, update, or delete.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: microservice
  name: cicd-role
rules:
- apiGroups: [""]
  resources: ["pods" , "services" , "secrets"]
  verbs: ["get", "list", "create", "watch" , "delete"]

- apiGroups: ["extensions" , "apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "delete"]
```

2. User or group:
   - In Kubernetes, users represent individuals or entities accessing the cluster.
   - groups are associated with teams. a group of users represents the `group` in k8s cluster.


3. Role Binding:
   - A role binding is a Kubernetes object that associates a role with a user, group, or service account, granting the permissions defined by the role to the associated subject.
   - Role bindings are created using the `RoleBinding` API object and defined in YAML manifests.
   - Role bindings establish the connection between roles and subjects, enabling the assignment of permissions.
   - Multiple role bindings can be created to associate multiple users, groups, or service accounts with a specific role.
   
## a scenario where user authentication is crucial in Kubernetes

In a multi-tenant Kubernetes cluster, there are multiple teams or organizations, each responsible for deploying and managing their own applications. User authentication becomes essential to ensure secure access control and proper separation of privileges.
