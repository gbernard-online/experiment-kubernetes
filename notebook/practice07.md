# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=ZGIfWYnodJQ&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=tF28iwTco9A&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #7 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

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

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-88cs4   1/1     Running   0          22s
nginx-7977cdf8f5-cb29x   1/1     Running   0          22s
nginx-7977cdf8f5-rxg88   1/1     Running   0          22s

$ kubectl create service nodeport nginx --dry-run=client --output=yaml --tcp=8080:80 |
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
  type: NodePort

$ kubectl apply --filename=service.yaml
service/nginx created

$ kubectl get services nginx --output=wide
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
nginx   NodePort   10.96.161.131   <none>        8080:30611/TCP   7s    app=nginx

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.161.131:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 127.0.0.1:30611
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ docker inspect --format='{{.NetworkSettings.Networks.kind.IPAddress}}' \
cluster-control-plane cluster-worker-green cluster-worker-red cluster-worker-yellow
172.17.1.2
172.17.1.3
172.17.1.4
172.17.1.5

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.2:30611
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.3:30611
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.4:30611
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.5:30611
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl get endpointslices.discovery.k8s.io nginx-kkx8s
NAME          ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
nginx-kkx8s   IPv4          80      10.244.3.27,10.244.2.33,10.244.1.29   2m55s

$ kubectl delete --filename=service.yaml
service "nginx" deleted

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml service.yaml
removed 'deployment.yaml'
removed 'service.yaml'
```

```bash
$ kubectl explain services.spec.externalName | cat --squeeze-blank
KIND:       Service
VERSION:    v1

FIELD: externalName <string>

DESCRIPTION:
    externalName is the external reference that discovery mechanisms will return
    as an alias for this service (e.g. a DNS CNAME record). No proxying will be
    involved.  Must be a lowercase RFC-1123 hostname
    (https://tools.ietf.org/html/rfc1123) and requires `type` to be
    "ExternalName".

$ kubectl create service externalname ipinfo --dry-run=client --external-name=ipinfo.io --output=yaml |
yq 'del(.metadata.labels) | del(.spec.selector)' | kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ipinfo
spec:
  externalName: ipinfo.io
  type: ExternalName

$ kubectl apply --filename=service.yaml
service/ipinfo created

$ kubectl get services ipinfo --output=wide
NAME     TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
ipinfo   ExternalName   <none>       ipinfo.io     <none>    6s    <none>

$ kubectl run alpine --image=alpine:latest --quiet --rm --restart=Never --stdin --tty -- \
wget -O - -q -T 5 ipinfo.io/org
AS12322 Free SAS

$ kubectl run alpine --image=alpine:latest --quiet --rm --restart=Never --stdin --tty -- \
nslookup ipinfo.default.svc.cluster.local | cat --squeeze
Server:		10.96.0.10
Address:	10.96.0.10:53

ipinfo.default.svc.cluster.local	canonical name = ipinfo.io
Name:	ipinfo.io
Address: 34.117.59.81

ipinfo.default.svc.cluster.local	canonical name = ipinfo.io

$ kubectl run alpine --image=alpine:latest --quiet --rm --restart=Never --stdin --tty -- \
wget -O - -q -T 5 ipinfo.default.svc.cluster.local/org
wget: server returned error: HTTP/1.1 404 Not Found
pod default/alpine terminated (Error)

$ kubectl run alpine --image=alpine:latest --quiet --rm --restart=Never --stdin --tty -- \
wget --header 'Host: ipinfo.io' -O - -q -T 5 ipinfo.default.svc.cluster.local/org
AS12322 Free SAS

$ kubectl delete --filename=service.yaml
deployment.apps "ipinfo" deleted

$ rm --verbose service.yaml
removed 'service.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
