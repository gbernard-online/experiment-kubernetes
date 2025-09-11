# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top

## PRACTICE #3 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
```



```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=yaml | kubectl-neat
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx

$ kubectl run nginx --image=nginx:alpine
nginx

$ kubectl logs pods/nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/07/09 20:34:57 [notice] 1#1: using the "epoll" event method
2025/07/09 20:34:57 [notice] 1#1: nginx/1.29.0
2025/07/09 20:34:57 [notice] 1#1: built by gcc 14.2.0 (Alpine 14.2.0)
2025/07/09 20:34:57 [notice] 1#1: OS: Linux 5.15.0-143-generic
2025/07/09 20:34:57 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 65536:65536
2025/07/09 20:34:57 [notice] 1#1: start worker processes
2025/07/09 20:34:57 [notice] 1#1: start worker process 30
2025/07/09 20:34:57 [notice] 1#1: start worker process 31
2025/07/09 20:34:57 [notice] 1#1: start worker process 32
2025/07/09 20:34:57 [notice] 1#1: start worker process 33

$ kubectl get events --field-selector=involvedObject.name=nginx
LAST SEEN   TYPE     REASON      OBJECT      MESSAGE
48s         Normal   Scheduled   pod/nginx   Successfully assigned default/nginx to ubuntu
47s         Normal   Pulled      pod/nginx   Container image "nginx:alpine" already present on machine
47s         Normal   Created     pod/nginx   Created container: nginx
46s         Normal   Started     pod/nginx   Started container nginx

$ kubectl top node
NAME     CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
ubuntu   337m         8%       1618Mi          42%

$ kubectl top pod
NAME    CPU(cores)   MEMORY(bytes)
nginx   0m           4Mi

$ kubectl delete pods nginx
pod "nginx" deleted

```





&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
