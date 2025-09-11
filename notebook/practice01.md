# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete

https://www.youtube.com/watch?v=ElVwAByDkf4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap&index=10

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
NAME    READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          28s   10.1.243.202   ubuntu   <none>           <none>

$ kubectl get pods --no-headers --output=wide
nginx   1/1   Running   0     42s   10.1.243.202   ubuntu   <none>   <none>

$ curl --fail --show-error --silent 10.1.243.202
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
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
$ kubectl run alpine --image=alpine:latest --rm --stdin --tty -- ash
If you donʼt see a command prompt, try pressing enter.
/ # hostname 
alpine
/ # ls /sys/class/net
eth0  lo
/ # exit
Session ended, resume using 'kubectl attach alpine -c alpine -i -t' command when the pod is running
pod "alpine" deleted

$ kubectl get pods
No resources found in default namespace.
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
