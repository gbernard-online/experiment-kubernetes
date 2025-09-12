# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=ZGIfWYnodJQ&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=tF28iwTco9A&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #7 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

#2 TODO : NODEPORT

#3 EXTERNAL

```bash
$ kubectl create service externalname ipinfo --dry-run=client --external-name=ipinfo.io --output=yaml |  yq 'del(.metadata.labels) | del(.spec.selector)' | kubectl-neat | tee service.yaml
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
ipinfo   ExternalName   <none>       ipinfo.io    <none>    19s   <none>

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget -qO- ipinfo.io
{
  "ip": "82.64.133.21",
  "hostname": "82-64-133-21.subs.proxad.net",
  "city": "Montrichard",
  "region": "Centre",
  "country": "FR",
  "loc": "47.3431,1.1865",
  "org": "AS12322 Free SAS",
  "postal": "41400",
  "timezone": "Europe/Paris",
  "readme": "https://ipinfo.io/missingauth"
}pod "alpine" deleted

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

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget --header 'Host: ipinfo.io' -qO- ipinfo.default.svc.cluster.local
{
  "ip": "82.64.133.21",
  "hostname": "82-64-133-21.subs.proxad.net",
  "city": "Montrichard",
  "region": "Centre",
  "country": "FR",
  "loc": "47.3431,1.1865",
  "org": "AS12322 Free SAS",
  "postal": "41400",
  "timezone": "Europe/Paris",
  "readme": "https://ipinfo.io/missingauth"
}pod "alpine" deleted

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget --header 'Host: ipinfo.io' -qO- ipinfo
{
  "ip": "82.64.133.21",
  "hostname": "82-64-133-21.subs.proxad.net",
  "city": "Montrichard",
  "region": "Centre",
  "country": "FR",
  "loc": "47.3431,1.1865",
  "org": "AS12322 Free SAS",
  "postal": "41400",
  "timezone": "Europe/Paris",
  "readme": "https://ipinfo.io/missingauth"
}pod "alpine" deleted

$ kubectl delete --filename=service.yaml
deployment.apps "ipinfo" deleted

$ rm --verbose service.yaml
removed 'service.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
