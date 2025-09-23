# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity

https://www.youtube.com/watch?v=A0Q04eIg1kA&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #18 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity
KIND:       Pod
VERSION:    v1

FIELD: affinity <Affinity>

DESCRIPTION:
    If specified, the podʼs scheduling constraints
    Affinity is a group of affinity scheduling rules.
|...|

$ kubectl explain pods.spec.affinity.nodeAffinity
KIND:       Pod
VERSION:    v1

FIELD: nodeAffinity <NodeAffinity>

DESCRIPTION:
    Describes node affinity scheduling rules for the pod.
    Node affinity is a group of node affinity scheduling rules.
|...|

$ kubectl explain pods.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution --recursive
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
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/env",
  "value": [
    {
      "name": "KUBERNETES_CONTAINER_NAME",
      "value": "alpine"
    }
  ]
}
]' --override-type=json --replicas=3 |
kubectl-neat | tee deployment.yaml
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
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
```




&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
