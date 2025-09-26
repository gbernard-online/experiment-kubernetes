# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity

https://www.youtube.com/watch?v=A0Q04eIg1kA&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #18 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: affinity <Affinity>

DESCRIPTION:
    If specified, the podʼs scheduling constraints
    Affinity is a group of affinity scheduling rules.

FIELDS:
  nodeAffinity	<NodeAffinity>
    Describes node affinity scheduling rules for the pod.

  podAffinity	<PodAffinity>
    Describes pod affinity scheduling rules (e.g. co-locate this pod in the same
    node, zone, etc. as some other pod(s)).

  podAntiAffinity	<PodAntiAffinity>
    Describes pod anti-affinity scheduling rules (e.g. avoid putting this pod in
    the same node, zone, etc. as some other pod(s)).

$ kubectl explain pods.spec.affinity.nodeAffinity | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: nodeAffinity <NodeAffinity>

DESCRIPTION:
    Describes node affinity scheduling rules for the pod.
    Node affinity is a group of node affinity scheduling rules.

FIELDS:
  preferredDuringSchedulingIgnoredDuringExecution	<[]PreferredSchedulingTerm>
    The scheduler will prefer to schedule pods to nodes that satisfy the
    affinity expressions specified by this field, but it may choose a node that
    violates one or more of the expressions. The node that is most preferred is
    the one with the greatest sum of weights, i.e. for each node that meets all
    of the scheduling requirements (resource request, requiredDuringScheduling
    affinity expressions, etc.), compute a sum by iterating through the elements
    of this field and adding "weight" to the sum if the node matches the
    corresponding matchExpressions; the node(s) with the highest sum are the
    most preferred.

  requiredDuringSchedulingIgnoredDuringExecution	<NodeSelector>
    If the affinity requirements specified by this field are not met at
    scheduling time, the pod will not be scheduled onto the node. If the
    affinity requirements specified by this field cease to be met at some point
    during pod execution (e.g. due to an update), the system may or may not try
    to eventually evict the pod from its node.

$ kubectl explain pods.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution \
KIND:       Pod
VERSION:    v1

FIELD: requiredDuringSchedulingIgnoredDuringExecution <NodeSelector>

DESCRIPTION:
    If the affinity requirements specified by this field are not met at
    scheduling time, the pod will not be scheduled onto the node. If the
    affinity requirements specified by this field cease to be met at some point
    during pod execution (e.g. due to an update), the system may or may not try
    to eventually evict the pod from its node.
    A node selector represents the union of the results of one or more label
    queries over a set of nodes; that is, it represents the OR of the selectors
    represented by the node selector terms.

FIELDS:
  nodeSelectorTerms	<[]NodeSelectorTerm> -required-
    matchExpressions	<[]NodeSelectorRequirement>
      key	<string> -required-
      operator	<string> -required-
      enum: DoesNotExist, Exists, Gt, In, ....
      values	<[]string>
    matchFields	<[]NodeSelectorRequirement>
      key	<string> -required-
      operator	<string> -required-
      enum: DoesNotExist, Exists, Gt, In, ....
      values	<[]string>
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=6 |
jq '.spec.template.spec.affinity = input' - <(echo '{
  "nodeAffinity": {
    "requiredDuringSchedulingIgnoredDuringExecution": {
      "nodeSelectorTerms": [
        {
          "matchExpressions": [
            {
              "key": "color",
              "operator": "In",
              "values": [
                "green",
                "yellow"
              ]
            }
          ]
        }
      ]
    }
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
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - green
                - yellow
      containers:
      - image: nginx:alpine
        name: nginx

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:green:yellow
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:green:yellow

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           14s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=6 |
jq '.spec.template.spec.affinity = input' - <(echo '{
  "nodeAffinity": {
    "requiredDuringSchedulingIgnoredDuringExecution": {
      "nodeSelectorTerms": [
        {
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
        {
          "matchFields": [
            {
              "key": "metadata.name",
              "operator": "In",
              "values": [
                "cluster-worker-yellow"
              ]
            }
          ]
        }
      ]
    }
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
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - green
            - matchFields:
              - key: metadata.name
                operator: In
                values:
                - cluster-worker-yellow
      containers:
      - image: nginx:alpine
        name: nginx

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:green:yellow
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:green:yellow

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           6s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-green

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=6 |
jq '.spec.template.spec.affinity = input' - <(echo '{
  "nodeAffinity": {
    "requiredDuringSchedulingIgnoredDuringExecution": {
      "nodeSelectorTerms": [
        {
          "matchExpressions": [
            {
              "key": "color",
              "operator": "In",
              "values": [
                "orange"
              ]
            }
          ]
        }
      ]
    }
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
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - orange
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
nginx   0/6     6            0           16s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].status.phase
Pending
Pending
Pending
Pending
Pending
Pending

$ kubectl patch deployments.apps nginx --patch='[
{
  "op": "replace",
  "path": "/spec/template/spec/affinity/nodeAffinity",
  "value": {
    "requiredDuringSchedulingIgnoredDuringExecution": {
      "nodeSelectorTerms": [
        {
          "matchExpressions": [
            {
              "key": "color",
              "operator": "In",
              "values": [
                "red"
              ]
            }
          ]
        }
      ]
    }
  }
}
]' --type=json
deployment.apps/nginx patched

$ kubectl get deployments.apps nginx --output=yaml | yq .spec.template.spec.affinity
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: color
            operator: In
            values:
              - red

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:red
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:orange
2         nginx:alpine:red

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   6/6     6            6           95s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].status.phase
Running
Running
Running
Running
Running
Running

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-red

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
