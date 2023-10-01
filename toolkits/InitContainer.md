
# InitContainer
The initContainer in Kubernetes is a special type of container that is executed before the main containers within a pod start running. Its purpose is to perform initialization tasks or prepare the environment for the primary containers.
The initContainer in Kubernetes is a special type of container that is executed before the main containers within a pod start running. Its purpose is to perform initialization tasks or prepare the environment for the primary containers. Here are a few key points about initContainer:

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
  replicas: 1
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
        ports:
          - containerPort: 80
        volumeMounts:
          - name: data
            mountPath: /usr/share/nginx/html
      - name: busybox
        image: busybox:latest
        command: ['sh' , '-c']
        args: ["while true; do echo $(date -u) 'this is a log , This is the Pod name $(POD_NAME)' >> /var/log/index.html; sleep 5;done"]
        volumeMounts:
          - name: data
            mountPath: /var/log
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
      initContainers:
      - name: initializer
        image: busybox:latest
        command: ['sh' , '-c']
        args: ["echo initialized the container ... > /var/log/index.html"]
        volumeMounts:
          - name: data
            mountPath: /var/log
      volumes:
        - name: data
          emptyDir: {}

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app.kubernetes.io/project: sample_project
spec:
  ports:
  - port: 80
  type: LoadBalancer
  selector:
    app.kubernetes.io/project: sample_project
```

```
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl get pod
NAME                  READY   STATUS     RESTARTS   AGE
test-bf64d588-t46zv   0/2     Init:0/1   0          3s
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl get pod
NAME                  READY   STATUS            RESTARTS   AGE
test-bf64d588-t46zv   0/2     PodInitializing   0          5s
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl get pod
NAME                  READY   STATUS            RESTARTS   AGE
test-bf64d588-t46zv   0/2     PodInitializing   0          6s
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl get pod
NAME                  READY   STATUS            RESTARTS   AGE
test-bf64d588-t46zv   0/2     PodInitializing   0          7s
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl get pod
NAME                  READY   STATUS    RESTARTS   AGE
test-bf64d588-t46zv   2/2     Running   0          8s
```

## The usage
Suppose you have a microservice-based application that consists of several components, including a web frontend, a backend API, and a database. The backend API depends on the database being ready before it can start processing requests. In this case, you can use an InitContainer to check the availability of the database service.
Once the InitContainer determines that the database service is available, the main container starts running and can safely assume that the necessary database dependency has been fulfilled.
By using an InitContainer in this scenario, you ensure that critical services or resources are ready before starting the main application container, reducing the chances of failures due to dependencies not being properly set up.
