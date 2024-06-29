# NodeAffinity
Node affinity in Kubernetes is a feature that allows you to define rules for pod placement based on characteristics of the nodes in your cluster. It determines which nodes are eligible for a particular pod to be scheduled on. Node affinity is used to influence or restrict the pod's placement on a node based on criteria such as node labels, node taints, or other node properties.

Node selector, on the other hand, is a simpler mechanism that leverages labels to specify the node requirements for pod scheduling. It allows you to assign a label to nodes and define a node selector in the pod specification to indicate which nodes are suitable for scheduling the pod.

Node affinity is preferred over node selector in some situations due to its flexibility and advanced matching capabilities. Here are a few reasons why node affinity may be better:

**1. Fine-grained control**: Node affinity provides more granular control over pod placement based on various node properties, including labels, taints, and other attributes. This allows for more specific criteria to be defined, giving greater control over pod scheduling.

**2. Complex node selection**: Node selector is limited to basic equality-based matching of labels, whereas node affinity supports a wider range of matching rules and expressions. This flexibility enables complex node selection logic, such as using set-based requirements or anti-affinity rules.

**3. Multi-zone deployments**: In scenarios where your cluster spans multiple availability zones or regions, node affinity becomes essential. It helps in spreading the workload across different zones by selecting nodes in specific zones or avoiding nodes in certain zones.

**4. Soft and hard preferences**: Node affinity allows you to specify both soft and hard preferences. Soft preferences indicate that a pod prefers but does not require a particular node, whereas hard preferences enforce strict requirements. This flexibility allows for better optimization of resource utilization within the cluster.

## Scenario
Let's consider a scenario where you have a Kubernetes cluster with there nodes. You want to schedule a set of pods that require GPU resources and you prefer to schedule them on SSD disks.
**what we have given to you :**
```
ali@ali-shazaei ~/k8s/scheduling $ kubectl get node
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   91m   v1.26.3
kind-worker          Ready    <none>          90m   v1.26.3
kind-worker2         Ready    <none>          91m   v1.26.3
kind-worker3         Ready    <none>          90m   v1.26.3
ali@ali-shazaei ~/k8s/scheduling $ 
ali@ali-shazaei ~/k8s/scheduling $ kubectl label node kind-worker gpu=yeah
node/kind-worker labeled
ali@ali-shazaei ~/k8s/scheduling $ kubectl label node kind-worker2 gpu=yeah
node/kind-worker2 labeled
ali@ali-shazaei ~/k8s/scheduling $ kubectl label node kind-worker2 disktype=ssd
node/kind-worker2 labeled
```
**Solution**
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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: gpu
                operator: In
                values:
                - yeah
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd

```
The **weight** field is an integer value that indicates how strongly the scheduler should prefer the nodes that match the given expressions. Higher values mean higher preference

In Kubernetes, node affinity uses operators to define rules for node selection. The main operators used in node affinity are:
- **In:** Matches nodes where the label value is within a specified list.
- **NotIn:** Matches nodes where the label value is not within a specified list.
- **Exists:** Matches nodes that have a specified label, regardless of value.
- **DoesNotExist:** Matches nodes that do not have the specified label.
- **Gt:** Matches nodes where the label value is greater than a specified value (for numeric values).
- **Lt:** Matches nodes where the label value is less than a specified value (for numeric values).
