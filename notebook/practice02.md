# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_explain

https://www.youtube.com/watch?v=ElVwAByDkf4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap&index=10
https://www.youtube.com/watch?v=MbQllww-qRw&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap&index=11  
https://www.youtube.com/watch?v=3iyMOfgZqAo&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap&index=13

## PRACTICE #2 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
```





```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure  -- sleep 10
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - sleep
    - "10"
    image: alpine:latest
    name: alpine
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: OnFailure
status: {}

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure -- sleep 10 | kubectl-neat | tee pod.yaml
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
    - "10"
    image: alpine:latest
    name: alpine
  restartPolicy: OnFailure

$ kubectl explain pods
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
...

$ kubectl explain pods.spec --recursive
KIND:       Pod
VERSION:    v1

FIELD: spec <PodSpec>


DESCRIPTION:
...

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl apply --filename=pod.yaml
pod/alpine unchanged

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          18s

$ kubectl get pods alpine --output=yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: db21901fc8440d95e170fe716b6d0e84346c9b8ad995478ffacc34dac5296350
    cni.projectcalico.org/podIP: ""
    cni.projectcalico.org/podIPs: ""
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"run":"alpine"},"name":"alpine","namespace":"default"},"spec":{"containers":[{"args":["sleep","10"],"image":"alpine:latest","name":"alpine"}],"restartPolicy":"OnFailure"}}
  creationTimestamp: "2025-07-09T16:30:33Z"
  labels:
    run: alpine
  name: alpine
  namespace: default
  resourceVersion: "8568"
  uid: c77045bd-4661-4ddd-be0e-076ee61c574b
...

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'

```





&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
