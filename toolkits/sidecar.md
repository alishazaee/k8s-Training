# SideCar
In Kubernetes, a sidecar is a design pattern where a secondary container is added to a pod alongside the main application container. The main purpose of a sidecar container is to enhance or extend the functionality of the main application container without directly modifying it. Sidecar containers run within the same pod and share the same network and storage resources.
Sidecar containers provide a flexible approach to extend and enhance the features and capabilities of an application without modifying it directly. They promote modular and decoupled architectures, making it easier to manage and scale complex applications in Kubernetes.
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
          args: ["while true; do echo $(date -u) 'this is a log' > /var/log/index.html; sleep 5;done"]
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

The Deployment section defines the deployment configuration for two containers: nginx and busybox. The nginx container uses the nginx:1.17 image and exposes port 80. It also mounts an emptyDir volume named "data" at /usr/share/nginx/html to store its data. The busybox container uses the busybox:latest image, runs a command to continuously echo a log message into a file every 5 seconds inside the /var/log directory, and also mounts the "data" volume. 

```
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
test   1/1     1            1           12m
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
test-6b6c774b97-q899x             2/2     Running   0          4m35s
ali@ali-shazaei ~/k8s/toolkits/sidecar $ kubectl logs  test-6b6c774b97-q899x -c nginx
```
As you can see, Both the nginx and busybox containers are defined within the same pod, allowing them to share the same resources and network namespace. This sharing of resources and close coupling between containers is a characteristic of sidecar pattern, where the sidecar container supports, enhances, or extends the functionality of the main application container.


## Sharing information between containers
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
          args: ["while true; do echo $(date -u) 'this is a log , This is the Pod name $(POD_NAME)' > /var/log/index.html; sleep 5;done"]
          volumeMounts:
            - name: data
              mountPath: /var/log
          env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name

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