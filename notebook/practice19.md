# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=zXCFyKc1_H4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #19 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity.nodeAffinity --output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: nodeAffinity <Object>

DESCRIPTION:
     Describes node affinity scheduling rules for the pod.

     Node affinity is a group of node affinity scheduling rules.

FIELDS:
   preferredDuringSchedulingIgnoredDuringExecution	<[]Object>
     The scheduler will prefer to schedule pods to nodes that satisfy the
     affinity expressions specified by this field, but it may choose a node that
     violates one or more of the expressions. The node that is most preferred is
     the one with the greatest sum of weights, i.e. for each node that meets all
     of the scheduling requirements (resource request, requiredDuringScheduling
     affinity expressions, etc.), compute a sum by iterating through the
     elements of this field and adding "weight" to the sum if the node matches
     the corresponding matchExpressions; the node(s) with the highest sum are
     the most preferred.
|...|

$ kubectl explain pods.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: preferredDuringSchedulingIgnoredDuringExecution <[]Object>

DESCRIPTION:
     The scheduler will prefer to schedule pods to nodes that satisfy the
     affinity expressions specified by this field, but it may choose a node that
     violates one or more of the expressions. The node that is most preferred is
     the one with the greatest sum of weights, i.e. for each node that meets all
     of the scheduling requirements (resource request, requiredDuringScheduling
     affinity expressions, etc.), compute a sum by iterating through the
     elements of this field and adding "weight" to the sum if the node matches
     the corresponding matchExpressions; the node(s) with the highest sum are
     the most preferred.

     An empty preferred scheduling term matches all objects with implicit weight
     0 (i.e. itʼs a no-op). A null preferred scheduling term matches no objects
     (i.e. is also a no-op).

FIELDS:
   preference	<Object> -required-
     A node selector term, associated with the corresponding weight.

   weight	<integer> -required-
     Weight associated with matching the corresponding nodeSelectorTerm, in the
     range 1-100.

$ kubectl explain pods.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution.preference \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: preference <Object>

DESCRIPTION:
     A node selector term, associated with the corresponding weight.

     A null or empty node selector term matches no objects. The requirements of
     them are ANDed. The TopologySelectorTerm type implements a subset of the
     NodeSelectorTerm.

FIELDS:
   matchExpressions	<[]Object>
     A list of node selector requirements by nodeʼs labels.

   matchFields	<[]Object>
     A list of node selector requirements by nodeʼs fields.
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
        "weight": 50
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
            weight: 50
      containers:
      - image: nginx:alpine
        name: nginx

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
    "weight": 25
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
      weight: 50
    - preference:
        matchFields:
          - key: metadata.name
            operator: In
            values:
              - cluster-worker-yellow
      weight: 25

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
        "weight": 50
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
            weight: 50
      containers:
      - image: nginx:alpine
        name: nginx

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
