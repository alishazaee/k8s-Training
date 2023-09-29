# User Authentication
each developer team works in a different namespace. we want to restrict access to only their namespaces so they don't accidentally break something in the namespace of the other team. we use RBAC or role based access control to do this.
## A scenario where user authentication is crucial in Kubernetes
In a multi-tenant Kubernetes cluster, there are multiple teams or organizations, each responsible for deploying and managing their own applications. User authentication becomes essential to ensure secure access control and proper separation of privileges.


In Kubernetes, the concepts of roles, role bindings, and users are interconnected to define access and permissions within the cluster. Here's an explanation of these concepts and their relationship:

1. Role:
   - A role is a Kubernetes object that defines a set of rules and permissions for accessing resources within a specific namespace.
   - Roles are created using the `Role` API object and defined in YAML manifests.
   - Roles can specify permissions for various resources, such as pods, services, secrets, config maps, etc.
   - Roles define what actions a user or service account can perform on the specified resources, like create, read, update, or delete.
this is a sample role configuration.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: microservice
  name: user-role
rules:
- apiGroups: [""]
  resources: ["pods" , "services" , "secrets"]
  verbs: ["get", "list", "create", "watch" , "delete"]

- apiGroups: ["extensions" , "apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "delete"]
```

2. User or group:
   - In Kubernetes, users represent individuals or entities accessing the cluster.
   - Users are not built in components in k8s. this means that k8s does not manage Users natively and no k8s objects are exist for representing normal user accounts. Users can be defined with static token file or certificates or even 3rd party identity service like LDAP.


3. Role Binding:
   - A role binding is a Kubernetes object that associates a role with a user, group, or service account, granting the permissions defined by the role to the associated subject.
   - Role bindings are created using the `RoleBinding` API object and defined in YAML manifests.
   - Role bindings establish the connection between roles and subjects, enabling the assignment of permissions.
   - Multiple role bindings can be created to associate multiple users, groups, or service accounts with a specific role.
   
## User Access Scenario
You have to create clinet keys for the user. the user will use these key pairs to authenticate with k8s. first you need to create these key pairs and CSR. 
In this case, we simply generate RSA private key for alishazaee user.
```
ali@ali-shazaei ~/temp $ openssl genrsa -out alishazaee.key 2048
ali@ali-shazaei ~/temp $ cat alishazaee.key 
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCMK8A6fmEOP+D+
PIuM3CKCOYF4/1GfZmGcENoaCOyQ/2C4/StI4JHAjXXT8tU4xDWIGMH7dCLBDT22
0gSEMsm7OdUjMoU880/stdco/zTTN+T1L3PnbAVKKUe7ctIPI3nfdrXcsWs74KJ6
TYnZLp63DWyBaFcxOLt9Acx5XdQ6ID211DF39VOHtZq/d9U4Az/yiJIwpHbFzASv
cccfDzouMUqe5aEP25NTG4lcGACkbijSXDvCuPKOfDUtdV8bfcm/d42ekBdvfSpO
Xg9sRTI+EhIYTknvj6XBnEdUarKdIOyFO9iO9rZR1iIHxHHRFXBQPbLTXgh8obW0
Q4p7AmXjAgMBAAECggEANJ61/p9z2vATDA35i0sWb1GcL6qVXyQFj5tp7O0dtb8Q
9dEgny6iuKjCK2tVLQbHW8yDgdySsWtBmDCWPnR8c5sdzqVIDF4Ayolm+L55e7NX
cc27EozkHXeKtK1BeypjtYZwdiVqbqOBCy2xioTsUyaobZoYZWN5Ss/SjVsycREc
KKFOcz4bgVkDkbPk8OW9V16R+ZWcE9CJGC588CU9w4DR62OkwhQHrdB7AOmc2Qbo
A/xuFeVI3CHpYl+VmgRXh1RAoqzM6hbxzQkM7shBrBB28Dk4c62ZRb08vVp6zweF
z9/muItNyt/mkHr0OF+0I5Gj3Tn6/uo05QztwiptNQKBgQDBMVT38X41K8T0/4Vp
U6y+73h57GbNGeKj/oDDsb6RfZycDblt7JrANl0XpxhjvFgTeI86YQ/0LOO55btv
Fha0sEDUcKnx0V/uhgZR3f6/6/FXgAx2+xeilAHl1MwwRXuXG45rSj5L8unefmnL
mZzFPjM1xjp81qbsE6gwzLMw/wKBgQC5vZ+MY3X2BIdkXBYVqgdzh9OusPjBmC0w
4Y9fZOfV3h/jA3h7BXxGBBt8gk1MGgT0oclHjWvtOTg8eE3x5fKVGL9Xj7SnCq5O
sdeuUuz9zW+ozKruzmJj8f16Mzmtvk7HD+mjD1m/musv/xDGEyLmti8ZyjpuEmxQ
IXWV8sEnHQKBgB4adcb81lGqtFIIzt0OsMg/wGIfOBWVhv9O1PmpZKx/Cjw210IN
sD9rOS5KVz6TRpYiHw9VuIqvw/xfk8lHg9o77J4twA1yNqSQNcPj96IoPb8IsOiJ
T7GBppoNgpOzAAMXxCVruDFVdKO6xvl2wjrp6kjizpJNUE1Q8tBH1VQ5AoGBALQt
ZBY1bXqHjicmxS2i0LObsRanCcgSrNPcGs54/gQTA2+eMEN2YMUyus8fP4hxPRlp
z+0fHPD0Lr9KHKJpY9aKOSLhfmcED267SfQK8WaK4KQoVjBt+DfnyPG/u1X1ZEnp
/8Rz4aXiy/61OTpL2fFgDXTBHcklCfj5XC1nXUNpAoGBAL1QQ+Fg3j/JTHKmOAo1
6E2ZRw/cNidTIZkvQbMlzuMPh5C7XB/tNmORps1rgVzRxdWAlwe5edfNldtDWSEM
wozTs28UyKgMI1GNwJCUaKUBXACsN7vd0/fgS8lkF/dDi3CdJ/Lot8nXSJaKOdtD
JRBgGtbiL33rRBNsdUhM9u7H
-----END PRIVATE KEY-----
```

kubernetes has certificate API which allows you to send it the CSR and k8s will sign it for us. It provides flexibility for generating and distributing certificates based on custom requirements specific to the applications running in the cluster.
the next step is to create CSR for this private key. we set certificate name to alishazaee which indicates that the certificate is for alishazaee user.
```
ali@ali-shazaei ~/temp $ openssl req -new -key alishazaee.key -subj "/CN=alishazaee" -out alishazaee.csr 
ali@ali-shazaei ~/temp $ cat alishazaee.csr 
-----BEGIN CERTIFICATE REQUEST-----
MIICWjCCAUICAQAwFTETMBEGA1UEAwwKYWxpc2hhemFlZTCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAIwrwDp+YQ4/4P48i4zcIoI5gXj/UZ9mYZwQ2hoI
7JD/YLj9K0jgkcCNddPy1TjENYgYwft0IsENPbbSBIQyybs51SMyhTzzT+y11yj/
NNM35PUvc+dsBUopR7ty0g8jed92tdyxazvgonpNidkunrcNbIFoVzE4u30BzHld
1DogPbXUMXf1U4e1mr931TgDP/KIkjCkdsXMBK9xxx8POi4xSp7loQ/bk1MbiVwY
AKRuKNJcO8K48o58NS11Xxt9yb93jZ6QF299Kk5eD2xFMj4SEhhOSe+PpcGcR1Rq
sp0g7IU72I72tlHWIgfEcdEVcFA9stNeCHyhtbRDinsCZeMCAwEAAaAAMA0GCSqG
SIb3DQEBCwUAA4IBAQBg3pJsBzawZDHdlHOao+nd4J0tQ+VWRTzzbyzotkMs5jQI
b2tki6/AUExvTa/C6gXjQoYOx1rUmaxxQQd4BX4pd4nPs9f50ZR5dUBbBZkHA4R6
spXXrL2fStm+q3uwYxvSBmcHekmzB0MtQXacWfCRX2Zv4gEfytWkflaEQE0jcIAL
69/3064AReO2IeolG3ZSXn2OsmQTSPJ+GIZSloMFdfTaVDE1u3Y3opRoP+UzEuNi
q3Jm5xuuWC1QLu+FRDC9v2VE2rmo6YRBowJDsE2IJozxtBDH5WJeDbQCFy1OYaV7
RHnv3MZ7DVOY5xZBFGANDEWKfDvB5xcVGbvWaGZH
-----END CERTIFICATE REQUEST-----
```
we need to send this `alishazaee.csr` to k8s so that k8s sign it for us using the k8s CA. the native way for doing this is using the k8s configuration file.
```
ali@ali-shazaei ~/temp $ cat alishazaee.csr | base64 | tr -d "\n" 
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dqQ0NBVUlDQVFBd0ZURVRNQkVHQTFVRUF3d0tZV3hwYzJoaGVtRmxaVENDQVNJd0RRWUpLb1pJaHZjTgpBUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSXdyd0RwK1lRNC80UDQ4aTR6Y0lvSTVnWGovVVo5bVlad1EyaG9JCjdKRC9ZTGo5SzBqZ2tjQ05kZFB5MVRqRU5ZZ1l3ZnQwSXNFTlBiYlNCSVF5eWJzNTFTTXloVHp6VCt5MTF5ai8KTk5NMzVQVXZjK2RzQlVvcFI3dHkwZzhqZWQ5MnRkeXhhenZnb25wTmlka3VucmNOYklGb1Z6RTR1MzBCekhsZAoxRG9nUGJYVU1YZjFVNGUxbXI5MzFUZ0RQL0tJa2pDa2RzWE1CSzl4eHg4UE9pNHhTcDdsb1EvYmsxTWJpVndZCkFLUnVLTkpjTzhLNDhvNThOUzExWHh0OXliOTNqWjZRRjI5OUtrNWVEMnhGTWo0U0VoaE9TZStQcGNHY1IxUnEKc3AwZzdJVTcySTcydGxIV0lnZkVjZEVWY0ZBOXN0TmVDSHlodGJSRGluc0NaZU1DQXdFQUFhQUFNQTBHQ1NxRwpTSWIzRFFFQkN3VUFBNElCQVFCZzNwSnNCemF3WkRIZGxIT2FvK25kNEowdFErVldSVHp6Ynl6b3RrTXM1alFJCmIydGtpNi9BVUV4dlRhL0M2Z1hqUW9ZT3gxclVtYXh4UVFkNEJYNHBkNG5QczlmNTBaUjVkVUJiQlprSEE0UjYKc3BYWHJMMmZTdG0rcTN1d1l4dlNCbWNIZWttekIwTXRRWGFjV2ZDUlgyWnY0Z0VmeXRXa2ZsYUVRRTBqY0lBTAo2OS8zMDY0QVJlTzJJZW9sRzNaU1huMk9zbVFUU1BKK0dJWlNsb01GZGZUYVZERTF1M1kzb3BSb1ArVXpFdU5pCnEzSm01eHV1V0MxUUx1K0ZSREM5djJWRTJybW82WVJCb3dKRHNFMklKb3p4dEJESDVXSmVEYlFDRnkxT1lhVjcKUkhudjNNWjdEVk9ZNXhaQkZHQU5ERVdLZkR2QjV4Y1ZHYnZXYUdaSAotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K

ali@ali-shazaei ~/temp $ cat << EOF | kubectl apply -f - 
> apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: alishazaee
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dqQ0NBVUlDQVFBd0ZURVRNQkVHQTFVRUF3d0tZV3hwYzJoaGVtRmxaVENDQVNJd0RRWUpLb1pJaHZjTgpBUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSXdyd0RwK1lRNC80UDQ4aTR6Y0lvSTVnWGovVVo5bVlad1EyaG9JCjdKRC9ZTGo5SzBqZ2tjQ05kZFB5MVRqRU5ZZ1l3ZnQwSXNFTlBiYlNCSVF5eWJzNTFTTXloVHp6VCt5MTF5ai8KTk5NMzVQVXZjK2RzQlVvcFI3dHkwZzhqZWQ5MnRkeXhhenZnb25wTmlka3VucmNOYklGb1Z6RTR1MzBCekhsZAoxRG9nUGJYVU1YZjFVNGUxbXI5MzFUZ0RQL0tJa2pDa2RzWE1CSzl4eHg4UE9pNHhTcDdsb1EvYmsxTWJpVndZCkFLUnVLTkpjTzhLNDhvNThOUzExWHh0OXliOTNqWjZRRjI5OUtrNWVEMnhGTWo0U0VoaE9TZStQcGNHY1IxUnEKc3AwZzdJVTcySTcydGxIV0lnZkVjZEVWY0ZBOXN0TmVDSHlodGJSRGluc0NaZU1DQXdFQUFhQUFNQTBHQ1NxRwpTSWIzRFFFQkN3VUFBNElCQVFCZzNwSnNCemF3WkRIZGxIT2FvK25kNEowdFErVldSVHp6Ynl6b3RrTXM1alFJCmIydGtpNi9BVUV4dlRhL0M2Z1hqUW9ZT3gxclVtYXh4UVFkNEJYNHBkNG5QczlmNTBaUjVkVUJiQlprSEE0UjYKc3BYWHJMMmZTdG0rcTN1d1l4dlNCbWNIZWttekIwTXRRWGFjV2ZDUlgyWnY0Z0VmeXRXa2ZsYUVRRTBqY0lBTAo2OS8zMDY0QVJlTzJJZW9sRzNaU1huMk9zbVFUU1BKK0dJWlNsb01GZGZUYVZERTF1M1kzb3BSb1ArVXpFdU5pCnEzSm01eHV1V0MxUUx1K0ZSREM5djJWRTJybW82WVJCb3dKRHNFMklKb3p4dEJESDVXSmVEYlFDRnkxT1lhVjcKUkhudjNNWjdEVk9ZNXhaQkZHQU5ERVdLZkR2QjV4Y1ZHYnZXYUdaSAotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 864000  
  usages:
  - client auth
> EOF
certificatesigningrequest.certificates.k8s.io/alishazaee created
```
the following command will list all the signing requests that are in the cluster and their conditions. Pending sate means that the admin must accept and approve this user.
```
ali@ali-shazaei ~/temp $ kubectl get csr
NAME         AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
alishazaee   10m   kubernetes.io/kube-apiserver-client   kubernetes-admin   100d                Pending
```
next step is to approve the user CA.
```
ali@ali-shazaei ~/temp $ kubectl certificate approve alishazaee
certificatesigningrequest.certificates.k8s.io/alishazaee approved
ali@ali-shazaei ~/temp $ kubectl get csr
NAME         AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
alishazaee   12m   kubernetes.io/kube-apiserver-client   kubernetes-admin   100d                Approved,Issued
```
then we should extract our crt file from `kubectl get csr -o yaml | grep certificate:`
```
ali@ali-shazaei ~/temp $ echo LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM5akNDQWQ2Z0F3SUJBZ0lRZTlIaEMzWmlvb2Q3aGpZcytWVGd6VEFOQmdrcWhraUc5dzBCQVFzRkFEQVYKTVJNd0VRWURWUVFERXdwcmRXSmxjbTVsZEdWek1CNFhEVEl6TURreU9URXdNVFV5T1ZvWERUSTBNREV3TnpFdwpNVFV5T1Zvd0VURVBNQTBHQTFVRUF4TUdZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBCk1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOdHYxRm1FUVIKd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQzdndoYmUrWgo2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFM25vV210c1lvCnJuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlESWYxcExaejIKalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFdxb3RuVm1GbApMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCbzBZd1JEQVRCZ05WSFNVRUREQUtCZ2dyCkJnRUZCUWNEQWpBTUJnTlZIUk1CQWY4RUFqQUFNQjhHQTFVZEl3UVlNQmFBRkhHTVF4RU5pT1A1bUJDYXVhUjYKcVlUanJaYXVNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUJMMFI5VHVodGlVYXE4QndPMnBseEMwVEhzZmhUNApHdFltNk5SNy8xWUlyMHEybFBpTzFhYTlMdWl0cmdnZkR2bUxHbW9yRlNETEg0ZEhDRE11aithSnJwNlNDSXR6CnFzL1d3OUo1TTBQL1RoS3A2UzNGK3BQQjZ0T2Z6aUJGaFFQOEh6N2JwalV2RGpyWkhnUWEzNVNDQUxoaC9tbTYKandBcUxqdWFiU1NuR2ZJRmdYNkE0RGxDaHZxZmhxemozckxPY3N0OTMyT0toZTNUVmltbGo1dmRKN0E2VklQdApFQXRUWlJESks1ZXEydkFvVEo5VmFzbVkrQ3o2L25rQ3Z1V2Zqb2tydWNvS2RPU3BvUEJrVWVNbFM0VkhFb3JWCmo3V2prc3dSUjI1dWJmdlVaaFZpcXRKVTZRYktuUnJjS2VYN0pBUVp5b3RTYWdWRG9yb0RGKzhmCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K | base64 --decode  > alishazaee.crt
ali@ali-shazaei ~/temp $ ls 
alishazaee.crt  alishazaee.csr  alishazaee.key  
```

We use these certificates to create the kubeconfig file. We can directly put the base64 format of those certificates in the kubeconfig file and give it to the developers to use it to access the cluster, or we can use the reference of the private key and the crt file in the kubeconfig.
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1Ea3lOVEUzTURjek5Wb1hEVE16TURreU1qRTNNRGN6TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBT0tECjdSS1RBVzFtVXdGSDdXRDFxR3hsYzhFaEJLOWNVdHJjUnVxcWlQNzhsWnRxOG0xNDVaelByM0ZaQy95RG03N1oKVGNYOVVKcE1OUWRYQ0dzUXNBTjh3QVNDeUUyZC84NGM2bTZocHpQV0JEZW9zeUVRTDV2VlQ3S2ZxcFVqZDFMbgpHdWVzcnlTOHdmQm1pK0RzTmo1dTJWSk5PV3U0VzF6TWppYnBOcEp5UCs1WkprYk1oeDFkMnI5UkI4cmlWRzNKCng1Vld3d0NhVUt6eXVDQW1maEJkWVpsRTNPaGszUjNBMitORU40OWdPSGNoaGx4RlZFRkNKSVgzbkRxTURIYlQKMFdFN0RkeHVUVk5OMlUyYzVtYnRwbE1xSkUyQmdmaytYVXFabTNSZEtFQlg0QkVzNTgxNGJLVDlFVUtzaDlCdQpGMWViQTZJcnlubER1dFppdUJzQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZIR01ReEVOaU9QNW1CQ2F1YVI2cVlUanJaYXVNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBS0dTSkdWdlN6NEp5MHhLaUhZNwp3UWlrWjQvOEZJRGd1dlBFNTJVTGJDQ0ZCb2Y3RGsvWkJ6NlZlYlBQUGF1SFpCMDBWa1dVNXVIYzdnamlCQStWCmNDbzFMZGRva3pFYXpOeE9iV3N6MW9DaVZRYUlsZW10cmhwZEZ4b3J3RndGRy82TDhXdHc3My9HZ1BvekpiTFYKcGI3ejJtNGIyVjhha28xWkZmUmFOR0NJeWFwU2xtZzd4Um9kcHkxU3hadW9MVjhpQ3hOSlBKczBRL0pyTjgvWAo4NkYrNnZ4QThjTENReGM5SE9HMHRjNDFHSzhiR2tqL0V2NVQ1WkJpUWdDN0hSZ2MwWW02c0tXZXNpa3JkSDFNCnFTYUYzQVJwY2VHQmdBVEtEM0FiaFdLbkVTaEMxWDFTS1ZFMEU3UWlBTk9BVDVkbkxBYjhHazBoc1hnd1pqWU4KUFhRPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://127.0.0.1:46459
  name: kind
contexts:
- context:
    cluster: kind
    user: alishazaee
  name: alishazaee@kind
current-context: alishazaee@kind
kind: Config
preferences: {}
users:
- name: alishazaee
  user:
    client-certificate: alishazaee.crt 
    client-key: alishazaee.key
```
Now Obviously, the alishazaee user has already been created, but because it has not been bound to any ClusterRole or Role, we do not have access to any resources.
```
ali@ali-shazaei ~/temp $ kubectl get pod
Error from server (Forbidden): pods is forbidden: User "alishazaee" cannot list resource "pods" in API group "" in the namespace "default"
```
one way to validate that the permissions are set correctly, is to use the following command as the admin user. When you execute this command, it sends a request to the Kubernetes API server with the provided user identity and asks if the user has permission to perform the specified action. The API server then consults its configured RBAC (Role-Based Access Control) rules to determine if the user has the necessary permissions.
```
ali@ali-shazaei ~/temp $ kubectl auth  can-i create pod --as alishazaee
no
```
There are several ways to access this user. If we want to give a developer access to the entire cluster, we must use the `clusterrole` resource inside Kubernetes, but if we want to create specific access to a namespace, we must use `Role` resource.
![image](https://github.com/alishb80/k8s-Training/assets/53411387/6f6d7829-e435-415b-9be0-86f580fa96b7)
