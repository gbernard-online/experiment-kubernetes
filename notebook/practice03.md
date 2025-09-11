# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_scale  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_edit

https://www.youtube.com/watch?v=7gxiNW3nSW8&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #3 - KUBERNETES - MICROK8S - UBUNTU 24

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
nginx-5hvcg   1/1     Running   0          10s
nginx-dn97b   1/1     Running   0          10s
nginx-ttmlv   1/1     Running   0          10s

$ kubectl describe pods --selector=app=nginx | fgrep Controlled
Controlled By:  ReplicaSet/nginx
Controlled By:  ReplicaSet/nginx
Controlled By:  ReplicaSet/nginx

$ kubectl scale replicaset nginx --replicas=9
replicaset.apps/nginx scaled

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   9         9         9       2m26s

$ kubectl get pods --selector=app=nginx
NAME          READY   STATUS    RESTARTS   AGE
nginx-5hvcg   1/1     Running   0          2m34s
nginx-74w9p   1/1     Running   0          11s
nginx-7bqlw   1/1     Running   0          11s
nginx-dn97b   1/1     Running   0          2m34s
nginx-n747d   1/1     Running   0          11s
nginx-pzczl   1/1     Running   0          11s
nginx-qwnv4   1/1     Running   0          11s
nginx-s27h6   1/1     Running   0          11s
nginx-ttmlv   1/1     Running   0          2m34s

$ kubectl edit replicasets.apps nginx
replicaset.apps/nginx edited

$ kubectl scale replicaset nginx --replicas=0
replicaset.apps/nginx scaled

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   0         0         0       4m9s

$ kubectl get pods --selector=app=nginx
No resources found in default namespace.

$ kubectl delete 



```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
