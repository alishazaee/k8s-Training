# Taint

A taint key is a unique identifier for a taint, and it can have an associated value. The taint key-value pair helps in adding more specificity to the taint and allows precise scheduling of pods.
Taints have three main effects on scheduling:
**1. NoSchedule**: This effect prevents the scheduler from placing pods on nodes with matching taints unless the pod has a toleration for that taint.

**2. PreferNoSchedule**: This effect represents a preference for the scheduler to avoid placing non-tolerating pods on nodes with matching taints. However, scheduling is still possible if required.

**3. NoExecute**: This effect not only prevents pod scheduling but also evicts any existing pods that do not tolerate the taint from the node.


All the master nodes have this taint. that is done by **kubeadm** when initiating the cluster.
```
ali@ali-shazaei ~ $ kubectl describe node | grep -i taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>
```
You can specify a taint for a node very easily :
```
ali@ali-shazaei ~/k8s/scheduling $ kubectl taint node kind-worker stat=notready:NoSchedule
```


# Toleration
In Kubernetes, toleration refers to a mechanism that allows pods to be scheduled on nodes with specific taints. Taints are labels applied to nodes, indicating that certain pods should not be scheduled on them unless they have a matching toleration.

For example, let's say you have a node in your Kubernetes cluster labeled with a taint of "high-priority=true". If you want to schedule a pod on this node, you need to apply a toleration to the pod that matches the taint. Here's an example YAML configuration for a pod with a toleration:

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: myimage
  tolerations:
  - key: "high-priority"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

In this example, the *tolerations* section specifies a toleration for the pod. The toleration's key, operator, value, and effect fields are set to match the taint on the node. In this case, it means that the pod can be scheduled on a node with the taint "high-priority=true".

By using tolerations, you can manage the scheduling of pods and control which nodes they can be placed on, based on specific taints applied to those nodes.
I will explain each field of toleration section in more detail:
- Operator: The operator specifies how the toleration should be evaluated. There are three possible operators:

   - Equal (==): The value of the toleration key must be an exact match to the taint key on the node.
   - Exists: The toleration is valid as long as there is any taint on the node with a matching key, regardless of the value.
   - DoesNotExist: The toleration is valid only if there are no taints on the node with a matching key.

- Key: The key is the identifier of the taint that the pod wants to tolerate. It is usually set to the key field of the taint.

- Effect: The effect of the taint determines what scheduling actions are blocked on the node. There are three effects:

   - NoSchedule: Prevents the pod from being scheduled on nodes with a matching taint.
   - PreferNoSchedule: Indicates a preference to avoid scheduling the pod on nodes with a matching taint, but it's not mandatory.
   - NoExecute: Evicts any existing pods that don't tolerate the taint from the node.



let's consider a scenario which you have a node with special hardwares. You want to schedule pods that require the special hardware only on the high-performance node. one solution to this problem is using taints and tolerations.

```
ali@ali-shazaei ~/k8s/scheduling $ kubectl taint node kind-worker2 hw=special:NoSchedule
node/kind-worker2 tainted
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test
  namespace: default
  labels:
    app.kubernetes.io/env: production
    app.kubernetes.io/project: sample_project
spec:
  replicas: 6
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
      tolerations:
      - key : hw
        value: special
        effect: NoSchedule
        operator: Equal

```

if you want to force all the pods to be scheduled on the tainted node, you can use *nodename* attribute:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test
  namespace: default
  labels:
    app.kubernetes.io/env: production
    app.kubernetes.io/project: sample_project
spec:
  replicas: 6
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
      tolerations:
      - key : hw
        value: special
        effect: NoSchedule
        operator: Equal
      nodeName: kind-worker2
```
