# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle

https://www.youtube.com/watch?v=33HwfEGifyc&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #11 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

## PRACTICE #12 - MICROK8S - UBUNTU 24

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
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
]' --override-type=json --restart=OnFailure -- sleep 60 | yq 'with(.spec.containers; .[1]=.[0])' |
yq 'with(.spec.containers; (.[0].name,.[0].env[0].value)="alpine-1")' |
yq 'with(.spec.containers; (.[1].name,.[1].env[0].value)="alpine-2")' |
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
    - sleep
    - "60"
    env:
    - name: KUBERNETES_CONTAINER_NAME
      value: alpine-1
    image: alpine:latest
    name: alpine-1
  - args:
    - sleep
    - "60"
    env:
    - name: KUBERNETES_CONTAINER_NAME
      value: alpine-2
    image: alpine:latest
    name: alpine-2
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS              RESTARTS   AGE
alpine   0/2     ContainerCreating   0          3s

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   2/2     Running   0          7s

$ kubectl exec alpine --container=alpine-1 -- env | fgrep KUBERNETES_CONTAINER_NAME
KUBERNETES_CONTAINER_NAME=alpine-1

$ kubectl exec alpine --container=alpine-2 -- env | fgrep KUBERNETES_CONTAINER_NAME
KUBERNETES_CONTAINER_NAME=alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/2     Completed   0          70s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'
```


&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
