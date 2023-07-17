# Node Selector
In Kubernetes, scheduling based on NodeSelector is a way to control which nodes a particular pod can be scheduled onto. It allows you to define specific requirements or preferences for the nodes where your pods should run.

NodeSelector is achieved by adding a label to the nodes and then specifying that label as a requirement in the pod's specification. The scheduler matches the node labels with the pod's node selector requirements and schedules the pod onto a suitable node.

Here's how it works:

**1. Label Nodes**: First, you need to label your nodes with specific key-value pairs. For example, you can label some nodes as "environment=production" and others as "environment=development".
```
ali@ali-shazaei ~/k8s/scheduling $ kubectl get nodes --show-labels
NAME                 STATUS   ROLES           AGE   VERSION   LABELS
kind-control-plane   Ready    control-plane   29m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-control-plane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
kind-worker          Ready    <none>          28m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker,kubernetes.io/os=linux
kind-worker2         Ready    <none>          28m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker2,kubernetes.io/os=linux
kind-worker3         Ready    <none>          28m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker3,kubernetes.io/os=linux
```
```
ali@ali-shazaei ~/k8s/scheduling $ kubectl label node kind-worker3 disktype=ssd
node/kind-worker3 labeled
ali@ali-shazaei ~/k8s/scheduling $ kubectl get nodes --show-labels
NAME                 STATUS   ROLES           AGE   VERSION   LABELS
kind-control-plane   Ready    control-plane   31m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-control-plane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
kind-worker          Ready    <none>          30m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker,kubernetes.io/os=linux
kind-worker2         Ready    <none>          30m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker2,kubernetes.io/os=linux
kind-worker3         Ready    <none>          30m   v1.26.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=kind-worker3,kubernetes.io/os=linux
```

**2. Define Node Selector**: In your deployment specification, you can define a node selector using the **nodeSelector** field. This field specifies key-value pairs that must match the labels on the nodes for scheduling.
```
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
      nodeSelector:
        disktype: ssd
```
**3. Scheduler Matching**: When you create or update a pod, the scheduler looks for nodes that have labels matching the specified node selector. It then schedules the pod onto one of those matching nodes.

```
ali@ali-shazaei ~/k8s/scheduling $ kubectl get pod -o wide
NAME                    READY   STATUS              RESTARTS   AGE   IP       NODE           NOMINATED NODE   READINESS GATES
test-68499869bd-rhw4w   0/1     ContainerCreating   0          21s   <none>   kind-worker3   <none>           <none>
test-68499869bd-vhqx8   0/1     ContainerCreating   0          21s   <none>   kind-worker3   <none>           <none>
test-68499869bd-x4j26   0/1     ContainerCreating   0          21s   <none>   kind-worker3   <none>           <none>
```

For example, in this case you have nodeSelector: disktype=ssd, so it will only be scheduled onto nodes labeled with disktype=ssd. If no suitable node is available, the pod remains unscheduled until a matching node becomes available.

NodeSelector provides basic affinity rules for scheduling pods based on specific criteria such as hardware capabilities, network locality, or any other custom requirements defined by labels on nodes.

Note that NodeSelector is just one way to control scheduling in Kubernetes. There are other advanced techniques like Node Affinity and Pod Affinity/Anti-Affinity that provide more fine-grained control over scheduling decisions based on complex rules and constraints.
