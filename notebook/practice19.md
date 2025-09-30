# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=zXCFyKc1_H4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #19 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain \
pods.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution \
--output=plaintext-openapiv2
|...|

$ kubectl explain \
pods.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution.preference \
--output=plaintext-openapiv2
|...|
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=6 |
jq '.spec.template.spec.affinity = input' - <(echo '{
  "nodeAffinity": {
    "preferredDuringSchedulingIgnoredDuringExecution": [
      {
        "preference": {
          "matchExpressions": [
            {
              "key": "color",
              "operator": "In",
              "values": [
                "green"
              ]
            }
          ]
        },
        "weight": 80
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
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - preference:
              matchExpressions:
              - key: color
                operator: In
                values:
                - green
            weight: 80
      containers:
      - image: nginx:alpine
        name: nginx

$ kubeconform -verbose deployment.yaml
deployment.yaml - Deployment nginx is valid

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:green
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:green

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           23s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green

$ kubectl scale deployment nginx --replicas=12
deployment.apps/nginx scaled

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   12/12   12           12          40s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-green

$ kubectl scale deployment nginx --replicas=18
deployment.apps/nginx scaled

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          69s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
     13 cluster-worker-green
      3 cluster-worker-red
      2 cluster-worker-yellow

$ kubectl patch deployments.apps nginx --patch='[
{
  "op": "add",
  "path": "/spec/template/spec/affinity/nodeAffinity/preferredDuringSchedulingIgnoredDuringExecution/-",
  "value": {
    "preference": {
      "matchFields": [
        {
          "key": "metadata.name",
          "operator": "In",
          "values": [
            "cluster-worker-yellow"
          ]
        }
      ]
    },
    "weight": 40
  }
}
]' --type=json
deployment.apps/nginx patched

$ kubectl get deployments.apps nginx --output=yaml | yq .spec.template.spec.affinity
nodeAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
    - preference:
        matchExpressions:
          - key: color
            operator: In
            values:
              - green
      weight: 80
    - preference:
        matchFields:
          - key: metadata.name
            operator: In
            values:
              - cluster-worker-yellow
      weight: 40

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:green:yellow
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:green
2         nginx:alpine:green:yellow

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          113s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-red
cluster-worker-green
cluster-worker-green

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
     11 cluster-worker-green
      1 cluster-worker-red
      6 cluster-worker-yellow

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=6 |
jq '.spec.template.spec.affinity = input' - <(echo '{
  "nodeAffinity": {
    "preferredDuringSchedulingIgnoredDuringExecution": [
      {
        "preference": {
          "matchExpressions": [
            {
              "key": "color",
              "operator": "In",
              "values": [
                "orange"
              ]
            }
          ]
        },
        "weight": 80
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
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - preference:
              matchExpressions:
              - key: color
                operator: In
                values:
                - orange
            weight: 80
      containers:
      - image: nginx:alpine
        name: nginx

$ kubeconform -verbose deployment.yaml
deployment.yaml - Deployment nginx is valid

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:orange
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:orange

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           26s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].status.phase
Running
Running
Running
Running
Running
Running

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-green
cluster-worker-red
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
