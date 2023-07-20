# Liveness Probe
A liveness probe is used to check if a container is responding properly and performing as expected. If the probe fails, the container is considered unhealthy, and Kubernetes takes remedial actions, like restarting the container. 
A readiness probe is used to verify if a container is ready to serve traffic. It allows Kubernetes to determine when a container is able to receive requests from a load balancer or another service.

Here's an example of a liveness probe & readiness probe
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 15

```
In this example, the liveness probe is configured similarly to the readiness probe:
- **httpGet**: Sends an HTTP GET request to the root path ("/") of the container on port 80.
- **initialDelaySeconds**: Specifies a delay of 10 seconds before the first liveness probe is performed after the container starts.
- **periodSeconds**: Sets the interval of subsequent liveness probes to occur every 15 seconds.

this is another example of checking the health of redis container : 
```
apiVersion: v1
kind: Pod
metadata:
  name: redis-server
spec:
  containers:
  - name: redis
    image: redis:latest
    ports:
    - containerPort: 6379
    livenessProbe:
      tcpSocket:
        port: 6379
      initialDelaySeconds: 15
      periodSeconds: 20
      timeoutSeconds: 5
    readinessProbe:
      exec:
        command: 
          - "redis-cli"
          - "ping"
      initialDelaySeconds: 15
      periodSeconds: 20
      timeoutSeconds: 5
```

