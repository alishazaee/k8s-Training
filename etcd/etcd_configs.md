
## etcd backup in k8s
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd-
cat /etc/kubernetes/manifests/etcd.yaml | grep server

/etc/kubernetes/pki/etcd/server.key
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/ca.crt
```

## etcd backup
```
NAME                 STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane   27m   v1.26.3   172.18.0.2    <none>        Ubuntu 22.04.2 LTS   5.15.0-56-generic   containerd://1.6.19-46-g941215f49
kind-worker          Ready    <none>          27m   v1.26.3   172.18.0.3    <none>        Ubuntu 22.04.2 LTS   5.15.0-56-generic   containerd://1.6.19-46-g941215f49
kind-worker2         Ready    <none>          27m   v1.26.3   172.18.0.4    <none>        Ubuntu 22.04.2 LTS   5.15.0-56-generic   containerd://1.6.19-46-g941215f49
kind-worker3         Ready    <none>          27m   v1.26.3   172.18.0.5    <none>        Ubuntu 22.04.2 LTS   5.15.0-56-generic   containerd://1.6.19-46-g941215f49
```
```
sudo ETCDCTL_ENDPOINTS=172.18.0.2:2379 ETCDCTL_API=3 etcdctl snapshot save snapshot.db   --cacert ca.crt   --cert server.crt   --key server.key
```

## checking the status of etcd backup 
```
ali@ali-shazaei ~/k8s/etcd $ sudo ETCDCTL_API=3 etcdctl snapshot status snapshot.db --write-out=table
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 7643f7a8 |     3449 |       1334 |     2.8 MB |
+----------+----------+------------+------------+
```

Now try to change the cluster. for example delete a deployment and restore the etcd by using the following command.
```
kubectl delete deployment nginx-deployment
```
```
sudo ETCDCTL_ENDPOINTS=172.18.0.2:2379 ETCDCTL_API=3 etcdctl snapshot restore snapshot.db   --data-dir /var/lib/minikube/etcd-backup 
```

Now modify the following line on master nodes. If you are using minikube, you can do it in /etc/kubernetes/manifests/etcd.yaml
```
  - hostPath:
      path: /var/lib/minikube/etcd-backup
      type: DirectoryOrCreate
```


You need to wait for a minite.
```
ali@ali-shazaei:~$ kubectl get pod 
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-96f55f99b-mftzl   1/1     Running   0          9h
nginx-deployment-96f55f99b-s9z67   1/1     Running   0          9h
```

if you have multiple master nodes, you have to restore etcd for every master node in your Kubernetes cluster. Each master node runs its own instance of etcd, which is responsible for storing the cluster configuration and state. Therefore, when restoring an etcd snapshot, you need to perform the restoration process on each master node to ensure that the etcd data is consistent across the cluster.

Restoring etcd on each master node ensures that all the control plane components, such as the kube-apiserver, kube-controller-manager, and kube-scheduler, have access to the same restored etcd data. This consistency is crucial for the proper functioning of the Kubernetes cluster and maintaining cluster integrity.
