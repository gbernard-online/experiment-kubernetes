# WIP: EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/configuration/configmap

https://www.youtube.com/watch?v=5b3kkJ0pUjA&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #9 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep configmaps
configmaps                          cm         v1                                true    ConfigMap

$ kubectl explain configmaps
KIND:       ConfigMap
VERSION:    v1

DESCRIPTION:
    ConfigMap holds configuration data for pods to consume.
|...|
```

```bash
$ kubectl create configmap alpine --dry-run=client --from-literal=DEFAULT_DELAY=30 --output=yaml |
kubectl-neat | tee configmap.yaml
apiVersion: v1
data:
  DEFAULT_DELAY: "30"
kind: ConfigMap
metadata:
  name: alpine

$ kubectl apply --filename=configmap.yaml
configmap/alpine created

$ kubectl get configmaps alpine
NAME     DATA   AGE
alpine   1      22s

$ kubectl describe configmaps alpine
Name:         alpine
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DEFAULT_DELAY:
----
30


BinaryData
====

Events:  <none>

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/env",
  "value": [
    {
      "name": "DELAY",
      "value": "60"
    }
  ]
},
{
  "op": "add",
  "path": "/spec/containers/0/envFrom",
  "value": [
    {
      "configMapRef": {
        "name": "alpine"
      }
    }
  ]
}
]' --override-type=json --restart=OnFailure -- ash -c 'sleep ${DELAY:-$DEFAULT_DELAY}' |
kubectl-neat | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - ash
    - -c
    - sleep ${DELAY:-$DEFAULT_DELAY}
    env:
    - name: DELAY
      value: "60"
    envFrom:
    - configMapRef:
        name: alpine
    image: alpine:latest
    name: alpine
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          14s

$ kubectl exec alpine -- env | fgrep DELAY
DEFAULT_DELAY=30
DELAY=60

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          89s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ kubectl delete --filename=configmap.yaml
configmap "alpine" deleted

$ rm --verbose configmap.yaml pod.yaml
removed 'configmap.yaml'
removed 'pod.yaml'
```

```bash
$ kubectl create configmap nginx --dry-run=client --from-file=nginx.conf=<(cat <<EOF
events{}
http {
  server {
    listen 80;
    location / {
      return 200 'OK\n';
      add_header Content-Type text/plain;
    }
  }
}
EOF
) --output=yaml | kubectl-neat | tee configmap.yaml
apiVersion: v1
data:
  nginx.conf: |
    events{}
    http {
      server {
        listen 80;
        location / {
          return 200 'OK\n';
          add_header Content-Type text/plain;
        }
      }
    }
kind: ConfigMap
metadata:
  name: nginx

$ kubectl apply --filename=configmap.yaml
configmap/nginx created

$ kubectl describe configmaps nginx
Name:         nginx
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx.conf:
----
events{}
http {
  server {
    listen 80;
    location / {
      return 200 'OK\n';
      add_header Content-Type text/plain;
    }
  }
}



BinaryData
====

Events:  <none>

$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/volumeMounts",
  "value": [
    {
      "name": "nginx",
      "mountPath": "/etc/nginx/nginx.conf",
      "subPath": "nginx.conf"
    }
  ]
},
{
  "op": "add",
  "path": "/spec/volumes",
  "value": [
    {
      "configMap": {
        "items": [
          {
            "key": "nginx.conf",
            "path": "nginx.conf"
          }
        ],
        "name": "nginx"
      },
      "name": "nginx"
    }
  ]
}
]' --override-type=json | kubectl-neat | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    volumeMounts:
    - mountPath: /etc/nginx/nginx.conf
      name: nginx
      subPath: nginx.conf
  volumes:
  - configMap:
      items:
      - key: nginx.conf
        path: nginx.conf
      name: nginx
    name: nginx

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl get pods --output=yaml | yq '.items | map({"name":.metadata.name,"ip":.status.podIP})'
- name: alpine
  ip: 10.244.1.3

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.244.1.3
OK

$ kubectl delete --filename=pod.yaml
pod "nginx" deleted

$ kubectl delete --filename=configmap.yaml
configmap "nginx" deleted

$ rm --verbose configmap.yaml pod.yaml
removed 'configmap.yaml'
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
