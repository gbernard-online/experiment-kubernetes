# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_api-resources  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_scale  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_edit

https://www.youtube.com/watch?v=7gxiNW3nSW8&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #3 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep replicasets
replicasets                         rs         apps/v1                           true    ReplicaSet

$ kubectl explain replicasets
GROUP:      apps
KIND:       ReplicaSet
VERSION:    v1

DESCRIPTION:
    ReplicaSet ensures that a specified number of pod replicas are running at
    any given time
|...|

$ kubectl create deployment nginx --dry-run=client --output=yaml --image=nginx:alpine --replicas=3 |
yq '.kind="ReplicaSet"' | kubectl-neat | tee replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx

$ kubectl apply --filename=replicaset.yaml
replicaset.apps/nginx created

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   3         3         0       5s

$ kubectl get pods --selector=app=nginx
NAME          READY   STATUS    RESTARTS   AGE
nginx-mmb2b   1/1     Running   0          15s
nginx-phw8v   1/1     Running   0          15s
nginx-tbkrh   1/1     Running   0          15s

$ kubectl describe pods --selector=app=nginx | fgrep Controlled
Controlled By:  ReplicaSet/nginx
Controlled By:  ReplicaSet/nginx
Controlled By:  ReplicaSet/nginx

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-yellow
cluster-worker-green

$ kubectl scale replicaset nginx --replicas=9
replicaset.apps/nginx scaled

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   9         9         9       68s

$ kubectl get pods --selector=app=nginx
NAME          READY   STATUS    RESTARTS   AGE
nginx-2bht7   1/1     Running   0          27s
nginx-7pwwh   1/1     Running   0          27s
nginx-dpc4t   1/1     Running   0          27s
nginx-h6bfs   1/1     Running   0          27s
nginx-lwq87   1/1     Running   0          27s
nginx-mmb2b   1/1     Running   0          80s
nginx-phw8v   1/1     Running   0          80s
nginx-tbkrh   1/1     Running   0          80s
nginx-wzz2r   1/1     Running   0          27s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-green
cluster-worker-red

$ kubectl edit replicasets.apps nginx
replicaset.apps/nginx edited

$ kubectl get replicasets.apps nginx --output=jsonpath='{.status.replicas}' && echo
6

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-red
cluster-worker-green

$ kubectl scale replicaset nginx --replicas=0
replicaset.apps/nginx scaled

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   0         0         0       2m43s

$ kubectl get pods --selector=app=nginx
No resources found in default namespace.

$ kubectl delete --filename=replicaset.yaml
replicaset.apps "nginx" deleted

$ rm --verbose replicaset.yaml
removed 'replicaset.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
