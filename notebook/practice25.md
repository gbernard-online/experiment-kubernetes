# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/pods/init-containers

## PRACTICE #25 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.initContainers --output=plaintext-openapiv2
|...|

$ kubectl explain pods.status.initContainerStatuses --output=plaintext-openapiv2
|...|

$ kubectl explain pods.status.initContainerStatuses.state --output=plaintext-openapiv2
|...|

$ kubectl explain pods.status.initContainerStatuses.state --output=plaintext-openapiv2 --recursive
|...|
```

```bash
$ kubectl run nginx --dry-run=client --image=nginx:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/initContainers",
  "value": [
    {
      "command": [
        "sleep",
        "30"
      ],
      "image": "alpine:latest",
      "name": "alpine-1"
    },
    {
      "command": [
        "sleep",
        "30"
      ],
      "image": "alpine:latest",
      "name": "alpine-2"
    }
  ]
}
]' --override-type=json --restart=OnFailure | kubectl-neat | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:latest
    name: nginx
  initContainers:
  - command:
    - sleep
    - "30"
    image: alpine:latest
    name: alpine-1
  - command:
    - sleep
    - "30"
    image: alpine:latest
    name: alpine-2
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl get pods nginx
NAME    READY   STATUS     RESTARTS   AGE
nginx   0/1     Init:0/2   0          10s

$ kubectl get pods nginx --output=yaml | yq .status.phase
Pending

$ kubectl get pods nginx --output=yaml | yq .status.initContainerStatuses[].state
running:
  startedAt: "2025-09-30T22:41:18Z"
waiting:
  reason: PodInitializing

$ kubectl get pods nginx
NAME    READY   STATUS     RESTARTS   AGE
nginx   0/1     Init:1/2   0          36s

$ kubectl get pods nginx --output=yaml | yq .status.phase
Pending

$ kubectl get pods nginx --output=yaml | yq .status.initContainerStatuses[].state
terminated:
  containerID: containerd://740061516c36af929a985118ef79f556dcd34cc6cb1050b2ae4623e87f20c967
  exitCode: 0
  finishedAt: "2025-09-30T22:41:48Z"
  reason: Completed
  startedAt: "2025-09-30T22:41:18Z"
running:
  startedAt: "2025-09-30T22:41:50Z"

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          81s

$ kubectl get pods nginx --output=yaml | yq .status.phase
Running

$ kubectl get pods nginx --output=yaml | yq .status.initContainerStatuses[].state
terminated:
  containerID: containerd://740061516c36af929a985118ef79f556dcd34cc6cb1050b2ae4623e87f20c967
  exitCode: 0
  finishedAt: "2025-09-30T22:41:48Z"
  reason: Completed
  startedAt: "2025-09-30T22:41:18Z"
terminated:
  containerID: containerd://710c33e32999a7e90d2a612acd3f339fdde1c46c795949e1e1872a2c46899d86
  exitCode: 0
  finishedAt: "2025-09-30T22:42:20Z"
  reason: Completed
  startedAt: "2025-09-30T22:41:50Z"

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
