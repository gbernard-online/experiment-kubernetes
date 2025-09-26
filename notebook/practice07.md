# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=ZGIfWYnodJQ&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=tF28iwTco9A&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #7 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

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
nginx-7977cdf8f5-dx8tg   1/1     Running   0          8s
nginx-7977cdf8f5-v4nhl   1/1     Running   0          8s
nginx-7977cdf8f5-vhzjp   1/1     Running   0          8s

$ kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed

$ kubectl get services nginx --output=wide
NAME    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE   SELECTOR
nginx   NodePort   10.96.130.30   <none>        80:31674/TCP   14s   app=nginx

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.130.30
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 127.0.0.1:31674
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ docker inspect cluster-control-plane --format='{{.NetworkSettings.Networks.kind.IPAddress}}'
172.17.1.3

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.3:31674
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl get endpointslices.discovery.k8s.io nginx-vx2nd
NAME          ADDRESSTYPE   PORTS   ENDPOINTS     AGE
nginx-vx2nd   IPv4          80      10.244.1.12   6m10s

$ kubectl delete services nginx
service "nginx" deleted

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted
```

```bash
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
nslookup ipinfo.default.svc.cluster.local
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
wget --header 'Host: ipinfo.io' -O - -q -T 5 ipinfo.default.svc.cluster.local/org | jq .org
AS12322 Free SAS

$ kubectl delete --filename=service.yaml
deployment.apps "ipinfo" deleted

$ rm --verbose service.yaml
removed 'service.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
