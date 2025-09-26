# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/pods  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_explain  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top

https://www.youtube.com/watch?v=ElVwAByDkf4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=MbQllww-qRw&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=3iyMOfgZqAo&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=naLtWR6uEWY&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #2 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24


```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

$ kubectl explain pods
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.
|...|

$ kubectl explain pods.spec --recursive
KIND:       Pod
VERSION:    v1

FIELD: spec <PodSpec>

DESCRIPTION:
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
    PodSpec is a description of a pod.
|...|

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
|...|

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

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure  -- sleep 10
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - sleep
    - "10"
    image: alpine:latest
    name: alpine
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: OnFailure
status: {}

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure -- sleep 10 |
kubectl-neat | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - sleep
    - "10"
    image: alpine:latest
    name: alpine
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl apply --filename=pod.yaml
pod/alpine unchanged

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          18s

$ kubectl get pods alpine --output=yaml
apiVersion: v1
kind: Pod
metadata:
|...|

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
