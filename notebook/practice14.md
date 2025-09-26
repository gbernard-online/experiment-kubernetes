# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node

https://www.youtube.com/watch?v=0KSOqB4nea0&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #14 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.nodeName
KIND:       Pod
VERSION:    v1

FIELD: nodeName <string>

DESCRIPTION:
    NodeName indicates in which node this pod is scheduled. If empty, this pod
    is a candidate for scheduling by the scheduler defined in schedulerName.
    Once this field is set, the kubelet for this node becomes responsible for
    the lifecycle of this pod. This field should not be used to express a desire
    for the pod to be scheduled on a specific node.
    https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename

$ kubectl explain pods.spec.nodeSelector
KIND:       Pod
VERSION:    v1

FIELD: nodeSelector <map[string]string>

DESCRIPTION:
    NodeSelector is a selector which must be true for the pod to fit on a node.
    Selector which must match a node's labels for the pod to be scheduled on
    that node. More info:
    https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
```

```bash
$ kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
cluster-control-plane   Ready    control-plane   10h   v1.34.0
cluster-worker-green    Ready    <none>          10h   v1.34.0
cluster-worker-red      Ready    <none>          10h   v1.34.0
cluster-worker-yellow   Ready    <none>          10h   v1.34.0

$ kubectl get nodes --output=json | jq '.items[].metadata.labels'
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/os": "linux",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "cluster-control-plane",
  "kubernetes.io/os": "linux",
  "node-role.kubernetes.io/control-plane": "",
  "node.kubernetes.io/exclude-from-external-load-balancers": ""
}
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/os": "linux",
  "color": "green",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "cluster-worker-green",
  "kubernetes.io/os": "linux"
}
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/os": "linux",
  "color": "red",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "cluster-worker-red",
  "kubernetes.io/os": "linux"
}
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/os": "linux",
  "color": "yellow",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "cluster-worker-yellow",
  "kubernetes.io/os": "linux"
}
```

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure -- sleep 60 |
kubectl-neat | yq '.spec.nodeName="cluster-worker-yellow" | sort_keys(.spec)' | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: alpine
  name: alpine
spec:
  containers:
    - args:
        - sleep
        - "60"
      image: alpine:latest
      name: alpine
  nodeName: cluster-worker-yellow
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeName}' && echo
cluster-worker-yellow

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          67s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/nodeSelector",
  "value": {
    "color": "red"
  }
}
]' --override-type=json --restart=OnFailure -- sleep 60 | kubectl-neat | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - sleep
    - "60"
    image: alpine:latest
    name: alpine
  nodeSelector:
    color: red
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeSelector}' && echo
{"color":"red"}

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeName}' && echo
cluster-worker-red

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          67s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=yaml --replicas=3 |
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

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           7s

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-26rt2   1/1     Running   0          28s
nginx-7977cdf8f5-59mn5   1/1     Running   0          28s
nginx-7977cdf8f5-d2l9d   1/1     Running   0          28s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-green
cluster-worker-yellow

$ kubectl patch deployments.apps nginx --patch='[
{
  "op": "add",
  "path": "/spec/template/spec/nodeSelector",
  "value": {
    "color": "green"
  }
}
]' --type=json
deployment.apps/nginx patched

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:green
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine
2         nginx:alpine:green

$ kubectl get pods --selector=app=nginx
NAME                    READY   STATUS    RESTARTS   AGE
nginx-c587dc8c6-2p2gc   1/1     Running   0          13s
nginx-c587dc8c6-l5dts   1/1     Running   0          18s
nginx-c587dc8c6-v69vp   1/1     Running   0          15s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-green

$ kubectl scale deployment nginx --replicas=9
deployment.apps/nginx scaled

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   9/9     9            9           117s

$ kubectl get pods --selector=app=nginx
NAME                    READY   STATUS    RESTARTS   AGE
nginx-c587dc8c6-2p2gc   1/1     Running   0          64s
nginx-c587dc8c6-4r5qn   1/1     Running   0          16s
nginx-c587dc8c6-7ph69   1/1     Running   0          16s
nginx-c587dc8c6-bwj9b   1/1     Running   0          16s
nginx-c587dc8c6-hzgm5   1/1     Running   0          16s
nginx-c587dc8c6-kzdns   1/1     Running   0          16s
nginx-c587dc8c6-l5dts   1/1     Running   0          69s
nginx-c587dc8c6-t5gc9   1/1     Running   0          16s
nginx-c587dc8c6-v69vp   1/1     Running   0          66s

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

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
