# Rolling Update

In a Rolling Update strategy in Kubernetes (k8s), old pods are gradually replaced with new and updated versions.

When implementing a Rolling Update, Kubernetes creates new pods with the updated version and keeps the old pods running. Gradually, one of the old pods is terminated and replaced with a new pod. This process is repeated until all the old pods have been replaced by the new ones.
![image](https://github.com/alishaza/k8s-Training/assets/53411387/4911c40c-2522-42af-9f64-50d0c92ccef8)

The benefits of using a Rolling Update strategy are:

-  **Continuous availability**: The service remains available during the update process since the old pods continue to function until the update is complete.

-  **Fault tolerance**: If the update includes any errors or issues, Kubernetes can automatically roll back to the previous state upon detecting a failure in the new pods.

-  **Speed control**: You can control the speed of the update by adjusting the number of replicas that are updated at a time. This allows you to limit resource impact and fine-tune your cluster capacity according to your needs.

![image](https://github.com/alishaza/k8s-Training/assets/53411387/cff1c203-3512-47a9-aef4-ef05fe43f3e5)


Rolling Update is a commonly used strategy in Kubernetes to minimize the impact of application updates, ensuring a seamless update process without downtime and maintaining high availability of the application.

this is a sample manifest file which uses rolling update strategy.
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
  replicas: 20
  # this scope defines the rolling update strategy 
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 1

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

-  **maxSurge**: This parameter specifies the maximum number (or percentage) of pods that can be created above the desired number of pods during the update. For example, if you have 10 replicas of a pod and set maxSurge to 2, Kubernetes can create up to 2 additional pods (above the desired 10) simultaneously. This helps to prevent any disruptions during the update by gradually introducing the new pods and ensuring sufficient capacity.

-  **maxUnavailable**: This parameter defines the maximum number (or percentage) of pods that can be unavailable during the update process. For instance, if you have 10 replicas of a pod and set maxUnavailable to 1, Kubernetes ensures that at least 9 out of the 10 pods remain available while replacing one pod at a time. This ensures the continuous availability of the application during the update.

By configuring these parameters appropriately, you can control the pace of the update, the availability of the application, and the impact on resources. This helps to achieve a smooth and controlled Rolling Update process in Kubernetes.


now try to update the vesion of nginx to 1.18 
```
kubectl edit deployment test
```
If you monitor the condition of the pods in advance, you will see that the pods are replaced little by little.
```
ali@ali-shazaei ~/k8s $ watch -n 0.1 kubectl get rs,pod,deployment
```
