# NodeName 
In Kubernetes (k8s), scheduling refers to the process of assigning pods (which are the basic units of deployment) to worker nodes in the cluster. By default, Kubernetes scheduler assigns pods to nodes in a balanced manner based on resource availability. However, scheduling can also be customized using various techniques.

Scheduling based on nodename is one such technique in Kubernetes that is not suitable for every situation. It allows you to specify a specific node where a pod should be scheduled.
suppose we want to schedule our pod on kind-worker2
```
ali@ali-shazaei ~/k8s/scheduling $ kubectl get node
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   8m46s   v1.26.3
kind-worker          Ready    <none>          8m7s    v1.26.3
kind-worker2         Ready    <none>          8m9s    v1.26.3
kind-worker3         Ready    <none>          8m7s    v1.26.3
```
this is the sample manifest that can be ysed to do this : 
```
kind: Deployment
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
      nodeName: kind-worker2
```
In the above example, the deployment named "test" will be scheduled only on the node with the name "kind-worker2". If the specified node is not available, the pod will remain in the pending state until the node becomes available or until a different node meets the scheduling requirements.

```
ali@ali-shazaei ~/k8s/scheduling $ kubectl get pod -o wide
NAME                   READY   STATUS              RESTARTS   AGE   IP       NODE           NOMINATED NODE   READINESS GATES
test-d587d7977-cmqm4   0/1     ContainerCreating   0          59s   <none>   kind-worker2   <none>           <none>
test-d587d7977-fx5vx   0/1     ContainerCreating   0          59s   <none>   kind-worker2   <none>           <none>
test-d587d7977-kwcp9   0/1     ContainerCreating   0          59s   <none>   kind-worker2   <none>           <none>
```


It's important to note that scheduling based on nodename limits the portability of your pods, as they become bound to a specific node. This approach is not good for cloud environments. In these environments worker names can change.
