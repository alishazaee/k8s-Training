#Deployments
### why to use deployments 
Deployments are used in Kubernetes to upgrade or downgrade services. In fact, deployment sits on top of replicaset and uses it behind the scenes. Deployment gives us the ability to roll out and roll back, and this allows us to create different strategies for new releases of our application.

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
```
now you can check the version of nginx

```
ali@ali-shazaei ~/k8s $ kubectl exec test-669648b5d-6spnz -- nginx -v
nginx version: nginx/1.17.10
```
## upgrading (roll out)
now it's time to roll out to nginx/1.18
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
  replicas: 10
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

new replicaset has been created and all pods got the new version.
```
ali@ali-shazaei ~/k8s $ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
test-669648b5d    0         0         0       14m
test-7d7b44bc55   10        10        10      5m42s
```

## downgrading (roll back)
Now, if we want to go back to the previous state, this is possible very easily. There may be a problem in the current version that we need to go back to the previous version
```
ali@ali-shazaei ~/k8s $ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
test-669648b5d    0         0         0       22m
test-7d7b44bc55   10        10        10      12m
ali@ali-shazaei ~/k8s $ kubectl rollout undo deployment test
deployment.apps/test rolled back
ali@ali-shazaei ~/k8s $ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
test-669648b5d    10        10        5       22m
test-7d7b44bc55   3         3         3       12m
ali@ali-shazaei ~/k8s $ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
test-669648b5d    10        10        10      22m
test-7d7b44bc55   0         0         0       13m
ali@ali-shazaei ~/k8s $ kubectl exec test-669648b5d-25dtv -- nginx -v
nginx version: nginx/1.17.10
```
## roll out to specific version
With every change that the deployment makes in the program version, a revision is made, and these revisions show the different releases of a program.
```
ali@ali-shazaei ~/k8s $ **kubectl rollout history deployment test**
deployment.apps/test 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
ali@ali-shazaei ~/k8s $ **kubectl rollout history deployment test --revision 2**
deployment.apps/test with revision #2
Pod Template:
  Labels:	app.kubernetes.io/project=sample_project
	pod-template-hash=7d7b44bc55
  Containers:
   nginx:
    Image:	**nginx:1.18**
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

ali@ali-shazaei ~/k8s $ **kubectl rollout undo deployment test --to-revision 2**
deployment.apps/test rolled back

```
This approach for upgrading or downgrading an application is not very useful in a production environment and this strategy is called **recreate**.
