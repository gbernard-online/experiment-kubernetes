# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=zXCFyKc1_H4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #20 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity.podAffinity --output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: podAffinity <Object>

DESCRIPTION:
     Describes pod affinity scheduling rules (e.g. co-locate this pod in the
     same node, zone, etc. as some other pod(s)).

     Pod affinity is a group of inter pod affinity scheduling rules.

FIELDS:
   preferredDuringSchedulingIgnoredDuringExecution	<[]Object>
     The scheduler will prefer to schedule pods to nodes that satisfy the
     affinity expressions specified by this field, but it may choose a node that
     violates one or more of the expressions. The node that is most preferred is
     the one with the greatest sum of weights, i.e. for each node that meets all
     of the scheduling requirements (resource request, requiredDuringScheduling
     affinity expressions, etc.), compute a sum by iterating through the
     elements of this field and adding "weight" to the sum if the node has pods
     which matches the corresponding podAffinityTerm; the node(s) with the
     highest sum are the most preferred.
|...

$ kubectl explain pods.spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: preferredDuringSchedulingIgnoredDuringExecution <[]Object>

DESCRIPTION:
     The scheduler will prefer to schedule pods to nodes that satisfy the
     anti-affinity expressions specified by this field, but it may choose a node
     that violates one or more of the expressions. The node that is most
     preferred is the one with the greatest sum of weights, i.e. for each node
     that meets all of the scheduling requirements (resource request,
     requiredDuringScheduling anti-affinity expressions, etc.), compute a sum by
     iterating through the elements of this field and subtracting "weight" from
     the sum if the node has pods which matches the corresponding
     podAffinityTerm; the node(s) with the highest sum are the most preferred.

     The weights of all of the matched WeightedPodAffinityTerm fields are added
     per-node to find the most preferred node(s)

FIELDS:
   podAffinityTerm	<Object> -required-
     Required. A pod affinity term, associated with the corresponding weight.

   weight	<integer> -required-
     weight associated with matching the corresponding podAffinityTerm, in the
     range 1-100.

$ kubectl explain \
pods.spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution.podAffinityTerm \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: podAffinityTerm <Object>

DESCRIPTION:
     Required. A pod affinity term, associated with the corresponding weight.

     Defines a set of pods (namely those matching the labelSelector relative to
     the given namespace(s)) that this pod should be co-located (affinity) or
     not co-located (anti-affinity) with, where co-located is defined as running
     on a node whose value of the label with key <topologyKey> matches that of
     any node on which a pod of the set of pods is running

FIELDS:
   labelSelector	<Object>
     A label query over a set of resources, in this case pods. If itʼs null,
     this PodAffinityTerm matches with no Pods.
|...|

   topologyKey	<string> -required-
     This pod should be co-located (affinity) or not co-located (anti-affinity)
     with the pods matching the labelSelector in the specified namespaces, where
     co-located is defined as running on a node whose value of the label with
     key topologyKey matches that of any node on which any of the selected pods
     is running. Empty topologyKey is not allowed.
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=3 |
jq '.spec.template.spec.affinity = input' - <(echo '{
  "podAffinity": {
    "preferredDuringSchedulingIgnoredDuringExecution": [
      {
        "podAffinityTerm": {
          "labelSelector": {
            "matchLabels": {
              "run": "redis"
            }
          },
          "topologyKey": "kubernetes.io/hostname"
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
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  run: redis
                topologyKey: kubernetes.io/hostname
            weight: 80
      containers:
      - image: nginx:alpine
        name: nginx

$ kubeconform -verbose deployment.yaml
deployment.yaml - Deployment nginx is valid

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
nginx   3/3     3            3           36s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-yellow
cluster-worker-green

$ kubectl run redis --image=redis:alpine
pod/redis created

$ kubectl get pods redis --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
redis   1/1     Running   0          12s   run=redis

$ kubectl get pods redis --output=yaml | yq .spec.nodeName
cluster-worker-red

$ kubectl scale deployment nginx --replicas=6
deployment.apps/nginx scaled

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-red
cluster-worker-green

$ kubectl scale deployment nginx --replicas=0
deployment.apps/nginx scaled

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/0     0            0           73s

$ kubectl scale deployment nginx --replicas=6
deployment.apps/nginx scaled

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           103s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red

$ kubectl scale deployment nginx --replicas=12
deployment.apps/nginx scaled

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   12/12   12           12          2m22s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-green
cluster-worker-red
cluster-worker-red
cluster-worker-red
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
