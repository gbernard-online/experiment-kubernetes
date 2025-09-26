# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete

https://www.youtube.com/watch?v=ElVwAByDkf4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #1 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl get pods
No resources found in default namespace.

$ kubectl run nginx --image=nginx:alpine
pod/nginx created

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          16s

$ kubectl get pods --output=wide
|...|

$ kubectl get pods nginx --output=yaml
apiVersion: v1
kind: Pod
metadata:
|...|

$ kubectl describe pods nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             ubuntu/172.16.0.225
|...|

$ kubectl get pods --output=custom-columns='NAME:.metadata.name,IP:.status.podIP,NODE:.spec.nodeName'
NAME    IP             NODE
nginx   10.1.243.202   ubuntu

$ kubectl get pods --output=yaml |
yq '.items | map({"name":.metadata.name,"ip":.status.podIP,"node":.spec.nodeName})'
- name: nginx
  ip: 10.1.243.202
  node: ubuntu

$ kubectl get pods --output=json |
jq '[.items[] | {"name":.metadata.name,"ip":.status.podIP,"node":.spec.nodeName}]'
[
  {
    "name": "nginx",
    "ip": "10.1.243.202",
    "node": "ubuntu"
  }
]

$ curl --connect-timeout 5 --fail --show-error --silent 10.1.243.202
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl delete pods nginx
pod "nginx" deleted
```

```bash
$ kubectl run alpine --image=alpine:latest --overrides='{"spec":{"restartPolicy":"OnFailure"}}' -- sleep 10
pod/alpine created

$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          18s

$ kubectl delete pods alpine
pod "alpine" deleted
```

```bash
$ kubectl run alpine --image=alpine:latest --quiet --rm --stdin --tty -- ash
/ # hostname
alpine
/ # exit

$ kubectl get pods
No resources found in default namespace.
```

## INSTALL - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24


```bash
$ kubectl get pods
No resources found in default namespace.

$ kubectl run nginx --image=nginx:alpine
pod/nginx created

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          8s

$ kubectl get pods --output=wide
|...|

$ kubectl get pods nginx --output=yaml
apiVersion: v1
kind: Pod
metadata:
|...|

$ kubectl describe pods nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             cluster-worker-green/172.17.1.4
|...|

$ kubectl get pods --output=custom-columns='NAME:.metadata.name,IP:.status.podIP,NODE:.spec.nodeName'
NAME    IP            NODE
nginx   10.244.3.11   cluster-worker-green

$ kubectl get pods --output=yaml |
yq '.items | map({"name":.metadata.name,"ip":.status.podIP,"node":.spec.nodeName})'
- name: nginx
  ip: 10.244.3.11
  node: cluster-worker-green

$ kubectl get pods --output=json |
jq '[.items[] | {"name":.metadata.name,"ip":.status.podIP,"node":.spec.nodeName}]'
[
  {
    "name": "nginx",
    "ip": "10.244.3.11",
    "node": "cluster-worker-green"
  }
]

$ curl --connect-timeout 5 --fail --show-error --silent 10.244.3.11
curl: (28) Connection timed out after 5002 milliseconds

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.244.3.11
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl delete pods nginx
pod "nginx" deleted
```

```bash
$ kubectl run alpine --image=alpine:latest --overrides='{"spec":{"restartPolicy":"OnFailure"}}' -- sleep 10
pod/alpine created

$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          7s

$ kubectl get pods
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          17s

$ kubectl delete pods alpine
pod "alpine" deleted
```

```bash
$ kubectl run alpine --image=alpine:latest --quiet --rm --stdin --tty -- ash
/ # hostname
alpine
/ # exit

$ kubectl get pods
No resources found in default namespace.
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
