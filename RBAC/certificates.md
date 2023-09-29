# Certificate Authority in K8s
In Kubernetes, certificates are signed by a Certificate Authority (CA). The CA can be either self-signed or from an external CA. kubeadm generates a CA for k8s cluster in /etc/kubernetes/pki folder.

By viewing the path `/etc/kubernetes/pki`, you can find the certificates and private/public keys related to the Kubernetes API server. Clients communicate with the API server using these credentials, which include:

1. apiserver.crt: This file contains the public key certificate for the API server.
2. apiserver.key: This file contains the private key for the API server.
3. ca.crt: This file contains the Certificate Authority (CA) certificate that is used to sign the certificates in the cluster.

The API server certificate verifies the identity of the server to the clients, while the private key allows the API server to decrypt client requests. The CA certificate is used by clients to verify the authenticity of the API server's certificate during TLS connection establishment.
```
ali@ali-shazaei ~ $ docker exec -it kind-control-plane ls /etc/kubernetes/pki
apiserver-etcd-client.crt     apiserver.key	  front-proxy-ca.key
apiserver-etcd-client.key     ca.crt		  front-proxy-client.crt
apiserver-kubelet-client.crt  ca.key		  front-proxy-client.key
apiserver-kubelet-client.key  etcd		  sa.key
apiserver.crt		      front-proxy-ca.crt  sa.pub
```
this is kubelet configuration file that contains necessary information for the kubelet to communicate securely with the Kubernetes API server and authenticate itself using client certificates and keys. we can see that CAs are defined in kubelet config file. In this case the clients can verify whether the server certificate was signed by it. in this way they can trust the communication. k8s CA's are trusted within k8s by all components, who have a copy of the CA. 
```
ali@ali-shazaei ~ $ docker exec -it kind-control-plane cat /etc/kubernetes/kubelet.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1Ea3lOVEUzTURjek5Wb1hEVE16TURreU1qRTNNRGN6TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBT0tECjdSS1RBVzFtVXdGSDdXRDFxR3hsYzhFaEJLOWNVdHJjUnVxcWlQNzhsWnRxOG0xNDVaelByM0ZaQy95RG03N1oKVGNYOVVKcE1OUWRYQ0dzUXNBTjh3QVNDeUUyZC84NGM2bTZocHpQV0JEZW9zeUVRTDV2VlQ3S2ZxcFVqZDFMbgpHdWVzcnlTOHdmQm1pK0RzTmo1dTJWSk5PV3U0VzF6TWppYnBOcEp5UCs1WkprYk1oeDFkMnI5UkI4cmlWRzNKCng1Vld3d0NhVUt6eXVDQW1maEJkWVpsRTNPaGszUjNBMitORU40OWdPSGNoaGx4RlZFRkNKSVgzbkRxTURIYlQKMFdFN0RkeHVUVk5OMlUyYzVtYnRwbE1xSkUyQmdmaytYVXFabTNSZEtFQlg0QkVzNTgxNGJLVDlFVUtzaDlCdQpGMWViQTZJcnlubER1dFppdUJzQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZIR01ReEVOaU9QNW1CQ2F1YVI2cVlUanJaYXVNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBS0dTSkdWdlN6NEp5MHhLaUhZNwp3UWlrWjQvOEZJRGd1dlBFNTJVTGJDQ0ZCb2Y3RGsvWkJ6NlZlYlBQUGF1SFpCMDBWa1dVNXVIYzdnamlCQStWCmNDbzFMZGRva3pFYXpOeE9iV3N6MW9DaVZRYUlsZW10cmhwZEZ4b3J3RndGRy82TDhXdHc3My9HZ1BvekpiTFYKcGI3ejJtNGIyVjhha28xWkZmUmFOR0NJeWFwU2xtZzd4Um9kcHkxU3hadW9MVjhpQ3hOSlBKczBRL0pyTjgvWAo4NkYrNnZ4QThjTENReGM5SE9HMHRjNDFHSzhiR2tqL0V2NVQ1WkJpUWdDN0hSZ2MwWW02c0tXZXNpa3JkSDFNCnFTYUYzQVJwY2VHQmdBVEtEM0FiaFdLbkVTaEMxWDFTS1ZFMEU3UWlBTk9BVDVkbkxBYjhHazBoc1hnd1pqWU4KUFhRPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://kind-control-plane:6443
  name: kind
contexts:
- context:
    cluster: kind
    user: system:node:kind-control-plane
  name: system:node:kind-control-plane@kind
current-context: system:node:kind-control-plane@kind
kind: Config
preferences: {}
users:
- name: system:node:kind-control-plane
  user:
    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
```


