# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=e4dpaiIEltk&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #23 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.topologySpreadConstraints --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=18 |
jq '.spec.template.spec.topologySpreadConstraints = input' - <(echo '[
  {
    "labelSelector": {
      "matchLabels": {
        "app": "nginx"
      }
    },
    "maxSkew": 1,
    "topologyKey": "color",
    "whenUnsatisfiable": "DoNotSchedule"
  }
]') | kubectl-neat --output=yaml  | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 18
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
      topologySpreadConstraints:
      - labelSelector:
          matchLabels:
            app: nginx
        maxSkew: 1
        topologyKey: color
        whenUnsatisfiable: DoNotSchedule

$ kubeconform -verbose deployment.yaml
deployment.yaml - Deployment nginx is valid

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:skew1
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:redis:skew1

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          19s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-green
cluster-worker-red

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      6 cluster-worker-green
      6 cluster-worker-red
      6 cluster-worker-yellow

$ kubectl get pods --output=yaml --selector=app=nginx |
yq '.items[] | select(.spec.nodeName=="cluster-worker-red").metadata.name' |
xargs --no-run-if-empty kubectl delete pods
pod "nginx-7cbb9b66db-4dt4d" deleted
pod "nginx-7cbb9b66db-d48pg" deleted
pod "nginx-7cbb9b66db-ljrsr" deleted
pod "nginx-7cbb9b66db-mqvjz" deleted
pod "nginx-7cbb9b66db-rrlzb" deleted
pod "nginx-7cbb9b66db-zb8dd" deleted

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          77s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-green
cluster-worker-green
cluster-worker-red
cluster-worker-yellow
cluster-worker-green
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-green

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      6 cluster-worker-green
      6 cluster-worker-red
      6 cluster-worker-yellow

$ kubectl patch deployments.apps nginx --patch='[
{
  "op": "replace",
  "path": "/spec/template/spec/topologySpreadConstraints/0/maxSkew",
  "value": 6
}
]' --type=json
deployment.apps/nginx patched

$ kubectl get deployments.apps nginx --output=yaml | yq .spec.template.spec.topologySpreadConstraints
- labelSelector:
    matchLabels:
      app: nginx
  maxSkew: 6
  topologyKey: color
  whenUnsatisfiable: DoNotSchedule

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:skew6
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:skew1
2         nginx:alpine:skew6

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          2m05s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-red

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      5 cluster-worker-green
      5 cluster-worker-red
      8 cluster-worker-yellow

$ kubectl get pods --output=yaml --selector=app=nginx |
yq '.items[] | select(.spec.nodeName=="cluster-worker-red").metadata.name' |
xargs --no-run-if-empty kubectl delete pods
pod "nginx-56d479d685-mc2hk" deleted
pod "nginx-56d479d685-mwvfh" deleted
pod "nginx-56d479d685-ng5xq" deleted
pod "nginx-56d479d685-sk4hn" deleted
pod "nginx-56d479d685-z57r5" deleted

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          2m45s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-yellow
cluster-worker-red
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-green

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      7 cluster-worker-green
      3 cluster-worker-red
      8 cluster-worker-yellow

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
