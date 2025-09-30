# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/services-networking/service

https://www.youtube.com/watch?v=Z62WCbIIWyg&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=ZGIfWYnodJQ&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #6 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep services | fgrep svc
services                            svc        v1                                true    Service

$ kubectl explain services --output=plaintext-openapiv2
|...|

$ kubectl explain services.spec --output=plaintext-openapiv2
|...|

$ kubectl explain services.spec.ports --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl create deployment nginx --image=nginx:alpine --replicas=3
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-bmzmc   1/1     Running   0          15s
nginx-7977cdf8f5-kg2l7   1/1     Running   0          15s
nginx-7977cdf8f5-t5kxh   1/1     Running   0          15s

$ kubectl expose deployment nginx --port=8080 --target-port=80
service/nginx exposed

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR
nginx   ClusterIP   10.96.46.237   <none>        8080/TCP   10s   app=nginx

$ curl --connect-timeout 5 --fail --show-error --silent 10.96.46.237:8080
curl: (28) Failed to connect to 10.96.46.237 port 8080 after 5002 ms: Timeout was reached

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.46.237:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
ash -c 'nslookup nginx.default.svc.cluster.local && cat /etc/resolv.conf' | cat --squeeze-blank
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	nginx.default.svc.cluster.local
Address: 10.96.46.237

search default.svc.cluster.local svc.cluster.local cluster.local qemu.lan
nameserver 10.96.0.10
options ndots:5

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx.default.svc.cluster.local:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl get endpointslices.discovery.k8s.io nginx-p6jj5
NAME          ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
nginx-p6jj5   IPv4          80      10.244.1.28,10.244.2.29,10.244.3.26   95s

$ kubectl scale deployment nginx --replicas=1
deployment.apps/nginx scaled

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-kg2l7   1/1     Running   0          2m39s

$ kubectl get endpointslices.discovery.k8s.io nginx-p6jj5
NAME          ADDRESSTYPE   PORTS   ENDPOINTS     AGE
nginx-p6jj5   IPv4          80      10.244.3.26   2m10s

$ kubectl delete services nginx
service "nginx" deleted

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=yaml --replicas=3 |
kubectl-neat | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
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
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl create service clusterip nginx --dry-run=client --output=yaml --tcp=8080:80 |
kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - name: 8080-80
    port: 8080
    targetPort: 80
  selector:
    app: nginx

$ kubectl apply --filename=service.yaml
service/nginx created

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR
nginx   ClusterIP   10.96.179.10   <none>        8080/TCP   15s   app=nginx

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.179.10:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl delete --filename=service.yaml
service "nginx" deleted

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml service.yaml
removed 'deployment.yaml'
removed 'service.yaml'
```

---

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
