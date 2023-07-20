# ReplicaSets and Pods
### APIs
You can use the following command to find out which api should be used to work with any resource in Kubernetes.

``` 
kubectl api-resources
```

for example you can get more help by using the following command : 

```
kubectl explain pods
```

## Pods
When working with Kubernetes, you don't have direct access to containers, and pods are an abstraction layer that manages containers.
```
apiVersion: v1
kind: Pod
metadata:
  name: test
  namespace: default
  labels:
    app.kubernetes.io/project: sample_project
    app.kubernetes.io/env: production
spec:
  containers:
    - name: nginx
      image: nginx:1.17
```
Using a pod means that if a problem occurs for that pod, it will not have HA capability


### Container Runtime
You can see your container runtime through the following command. In minikube, the container runtime is Docker, while in new versions of Kubernetes, Docker is no longer used as a container runtime.
```
ali@ali-shazaei ~/k8s $ kubectl get no -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube       Ready    control-plane   77d   v1.26.3   192.168.49.2   <none>        Ubuntu 20.04.5 LTS   5.15.0-56-generic   docker://23.0.2
minikube-m02   Ready    <none>          31m   v1.26.3   192.168.49.3   <none>        Ubuntu 20.04.5 LTS   5.15.0-56-generic   docker://23.0.2
```

## ReplicaSet
ReplicaSets have some benefits for us. In nutshell, these are the benefits that this resource can provide for us : 
- **Desired state** : The desired state is constantly compared with the current state, and if the two are different from each other, this resource takes action.
- **Scaling** : With this resource, you can increase or decrease the number of replicas and easily scale a service.
- **Self Healing** : This feature helps to automatically replace a pod if a problem occurs.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: test
  namespace: default
  labels:
    app.kubernetes.io/env: production
    app.kubernetes.io/project: sample_project
spec:
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/project: sample_project
  template:
    metadata:
      labels:
        app.kubernetes.io/project: sample_project
    spec:
      containers:
        - name: nginx
          image: nginx:1.17

```

### Scaling Capability 
By using the following command, you tell replicaset to always make sure that the number of my pods is not less than 7 from the desired template.
```
kubectl scale replicaset  test --replicas 7
```

### upgrading or downgrading
This resource is not responsible for upgrading or downgrading your program in any way. so in this scenario, suppose you want to upgrade your app to nginx:1.18. you can't do this by simply changing the templates.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: test
  namespace: default
  labels:
    app.kubernetes.io/env: production
    app.kubernetes.io/project: sample_project
spec:
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/project: sample_project
  template:
    metadata:
      labels:
        app.kubernetes.io/project: sample_project
    spec:
      containers:
        - name: nginx
          image: nginx:1.18

```
now if you check the vesion of your pod, the vesion has not changed.
```
ali@ali-shazaei ~/k8s $ kubectl exec test-89r4b -- nginx -v
nginx version: nginx/1.17
```
This change is applied when a new pod wants to be recreated from that template


