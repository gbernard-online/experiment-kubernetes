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
$ kubectl create deployment nginx --image=nginx:alpine --replicas=3
deployment.apps/nginx created

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-445s7   1/1     Running   0          24s
nginx-7977cdf8f5-s5xqr   1/1     Running   0          24s
nginx-7977cdf8f5-vpksn   1/1     Running   0          24s

$ kubectl get pods --output=yaml --selector=app=nginx |
yq '.items | map({"name":.metadata.name,"ip":.status.podIP,"node":.spec.nodeName})'
- name: nginx-7977cdf8f5-445s7
  ip: 10.244.2.11
  node: cluster-worker-yellow
- name: nginx-7977cdf8f5-s5xqr
  ip: 10.244.1.12
  node: cluster-worker-red
- name: nginx-7977cdf8f5-vpksn
  ip: 10.244.3.14
  node: cluster-worker-green

$ kubectl expose deployment nginx --port=80
service/nginx exposed

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx   ClusterIP   10.96.57.201   <none>        80/TCP    21s   app=nginx

$ curl --connect-timeout 5 --fail --show-error --silent 10.96.57.201 
curl: (28) Failed to connect to 10.152.183.244 port 80 after 5001 ms: Timeout was reached

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.57.201
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
ash -c 'nslookup nginx.default.svc.cluster.local && cat /etc/resolv.conf'
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	nginx.default.svc.cluster.local
Address: 10.96.57.201


search default.svc.cluster.local svc.cluster.local cluster.local qemu.lan
nameserver 10.96.0.10
options ndots:5

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -qO- nginx.default.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -qO- nginx
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl get endpointslices.discovery.k8s.io nginx-vx2nd 
NAME          ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
nginx-vx2nd   IPv4          80      10.244.2.11,10.244.3.14,10.244.1.12   4m1s

$ kubectl scale deployment nginx --replicas=1
deployment.apps/nginx scaled

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-s5xqr   1/1     Running   0          52m

$ kubectl get endpointslices.discovery.k8s.io nginx-vx2nd 
NAME          ADDRESSTYPE   PORTS   ENDPOINTS     AGE
nginx-vx2nd   IPv4          80      10.244.1.12   6m10s

$ kubectl delete services nginx
service "nginx" deleted

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted
```

---

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
