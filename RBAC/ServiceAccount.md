# Service Accounts (SA)

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
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitlab
  namespace: microservice
automountServiceAccountToken: true
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-rb
  namespace: microservice
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cicd-role
subjects:
- kind: ServiceAccount
  name: gitlab
  namespace: microservice
---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: gitlab-token
  namespace: microservice
  annotations:
    kubernetes.io/service-account.name: "gitlab"

```