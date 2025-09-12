# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=Z62WCbIIWyg&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=ZGIfWYnodJQ&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #6 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash






#TODO
DNS TO ADD


$ kubectl create deployment nginx --image=nginx:alpine --replicas=3
deployment.apps/nginx created

$ kubectl get pods --output=wide --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE    IP             NODE     NOMINATED NODE   READINESS GATES
nginx-6b66fbbd46-4dbgj   1/1     Running   0          119s   10.1.243.193   ubuntu   <none>           <none>
nginx-6b66fbbd46-4j8bg   1/1     Running   0          119s   10.1.243.237   ubuntu   <none>           <none>
nginx-6b66fbbd46-x4t45   1/1     Running   0          119s   10.1.243.195   ubuntu   <none>           <none>

$ kubectl expose deployment nginx --port=80
service/nginx exposed

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx   ClusterIP   10.152.183.244   <none>        80/TCP    9s    app=nginx

$ curl --fail --show-error --silent http://10.152.183.244
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ kubectl get endpoints nginx
NAME    ENDPOINTS                                         AGE
nginx   10.1.243.193:80,10.1.243.195:80,10.1.243.237:80   4m1s

$ kubectl scale deployment nginx --replicas=1
deployment.apps/nginx scaled

$ kubectl get endpoints nginx
NAME    ENDPOINTS         AGE
nginx   10.1.243.237:80   6m10s

$ kubectl scale deployment nginx --replicas=9
deployment.apps/nginx scaled

$ kubectl get endpoints nginx
NAME    ENDPOINTS         AGE
nginx   10.1.243.237:80   10m

$ kubectl get endpoints nginx
NAME    ENDPOINTS                                                     AGE
nginx   10.1.243.234:80,10.1.243.235:80,10.1.243.237:80 + 3 more...   10m

$ kubectl get endpoints nginx
NAME    ENDPOINTS                                                     AGE
nginx   10.1.243.234:80,10.1.243.235:80,10.1.243.236:80 + 6 more...   11m

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted

$ kubectl delete services nginx
service "nginx" deleted

```


LIKE THIS

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- nslookup ipinfo.default.svc.cluster.local
Server:		10.152.183.10
Address:	10.152.183.10:53

ipinfo.default.svc.cluster.local	canonical name = ipinfo.io

ipinfo.default.svc.cluster.local	canonical name = ipinfo.io
Name:	ipinfo.io
Address: 34.117.59.81

pod "alpine" deleted

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget -qO- ipinfo.default.svc.cluster.local
wget: server returned error: HTTP/1.1 404 Not Found
pod "alpine" deleted
pod default/alpine terminated (Error)

---

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
