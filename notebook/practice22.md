# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=e4dpaiIEltk&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #22 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity.podAntiAffinity --output=plaintext-openapiv2
|...|

$ kubectl explain pods.spec.affinity.podAntiAffinity --output=plaintext-openapiv2 --recursive
|...|
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=6 |
jq '.spec.template.spec.affinity = input' - <(echo '{
  "podAntiAffinity": {
    "requiredDuringSchedulingIgnoredDuringExecution": [
      {
        "labelSelector": {
          "matchLabels": {
            "run": "redis"
          }
        },
        "topologyKey": "kubernetes.io/hostname"
      }
    ]
  }
}') | kubectl-neat --output=yaml  | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 6
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                run: redis
            topologyKey: kubernetes.io/hostname
      containers:
      - image: nginx:alpine
        name: nginx

$ kubeconform -verbose deployment.yaml
deployment.yaml - Deployment nginx is valid

$ kubectl run redis --image=redis:alpine
pod/redis created

$ kubectl get pods redis --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
redis   1/1     Running   0          13s   run=redis

$ kubectl get pods redis --output=yaml | yq .spec.nodeName
cluster-worker-red

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:redis
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:redis

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           14s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ kubectl delete pods/redis
pod "redis" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
