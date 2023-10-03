# Tshoot Strategies
when we try to run a container using `docker run` we can simply override the `CMD` instructions by passing the alternative command as a parameter to `docker run` command.
`EntryPoint` instruction in docker allows you to append additional commands on top of the entrypoint. CMD is used when we want to use a default command that users can override. Entrypoint is used when want to define a specific executable that should always run.
In k8s we can replace the entrypoint instruction in dockerfile with an attribute called `command` and the CMD instruction can be replaced with `args`.
In the provided manifest below, both of them are defined which means the entrypoint is the command and the CMD is the arguments to that command.
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: test
    image: busybox
    command: ["sh"]
    args: ["-c" ,"sleep 1000"]

```
In many cases, we may want to check the connection between the pods in a cluster or detect a problem in our cluster. For example, in the following command, we create a busybox image and check the connection to the Internet. It also helps us when we want to check the connection with another pod.

```
ali@ali-shazaei ~/k8s/toolkits $ kubectl run test2 --image busybox --rm -it -- ping 8.8.8.8
If you don't see a command prompt, try pressing enter.
64 bytes from 8.8.8.8: seq=1 ttl=46 time=57.811 ms
64 bytes from 8.8.8.8: seq=2 ttl=46 time=56.846 ms
```

