# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/storage/volumes

https://www.youtube.com/watch?v=0KSOqB4nea0&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #12 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24


```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/volumeMounts",
  "value": [
    {
      "mountPath": "/cache",
      "name": "alpine"
    }
  ]
},
{
  "op": "add",
  "path": "/spec/volumes",
  "value": [
    {
      "name": "alpine"
    }
  ]
}
]' --override-type=json --restart=OnFailure -- sleep 120 |
yq 'with(.spec.containers; .[1]=.[0] | .[0].name="alpine-1" | .[1].name="alpine-2")' |
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
    - "120"
    image: alpine:latest
    name: alpine-1
    volumeMounts:
    - mountPath: /cache
      name: alpine
  - args:
    - sleep
    - "120"
    image: alpine:latest
    name: alpine-2
    volumeMounts:
    - mountPath: /cache
      name: alpine
  restartPolicy: OnFailure
  volumes:
  - name: alpine

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   2/2     Running   0          7s

$ kubectl get pods alpine --output=jsonpath='{.spec.volumes[0]}' && echo
{"emptyDir":{},"name":"alpine"}

$ kubectl get pods --output=yaml |
yq '.items | map({"name":.metadata.name,"node":.spec.nodeName})'
- name: alpine
  node: cluster-worker-red

$ docker exec cluster-worker-red find /var/lib/kubelet/pods -name 'alpine-*'
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/containers/alpine-1
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/containers/alpine-2

$ kubectl exec alpine --container=alpine-1 -- touch /cache/alpine-1

$ kubectl exec alpine --container=alpine-2 -- touch /cache/alpine-2

$ kubectl exec alpine --container=alpine-1 -- ls /cache
alpine-1
alpine-2

$ kubectl exec alpine --container=alpine-2 -- ls /cache
alpine-1
alpine-2

$ kubectl exec alpine --container=alpine-1 -- df -h /cache
Filesystem                Size      Used Available Use% Mounted on
/dev/vda2                30.3G     13.3G     15.4G  46% /cache

$ kubectl exec alpine --container=alpine-2 -- df -h /cache
Filesystem                Size      Used Available Use% Mounted on
/dev/vda2                30.3G     13.3G     15.4G  46% /cache

$ docker exec cluster-worker-red find /var/lib/kubelet/pods -name 'alpine-*'
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/containers/alpine-1
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/containers/alpine-2
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/volumes/kubernetes.io~empty-dir/alpine/alpine-1
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/volumes/kubernetes.io~empty-dir/alpine/alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/2     Completed   0          2m6s

$ docker exec cluster-worker-red find /var/lib/kubelet/pods -name 'alpine-*'
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/containers/alpine-1
/var/lib/kubelet/pods/e6239fea-a73d-4a68-8dda-d5777c1f01c8/containers/alpine-2

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ docker exec cluster-worker-red find /var/lib/kubelet/pods -name 'alpine-*'

$ docker exec cluster-worker-red ls /var/lib/kubelet/pods
f0338d9b-b37c-4537-888c-d2e5b83a6ebb
fdc41f67-343b-4c79-a89f-64fb035883a6

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/volumeMounts",
  "value": [
    {
      "mountPath": "/cache",
      "name": "alpine"
    }
  ]
},
{
  "op": "add",
  "path": "/spec/volumes",
  "value": [
    {
      "emptyDir":
      {
        "medium": "Memory",
        "sizeLimit": "1Mi"
      },
      "name": "alpine"
    }
  ]
}
]' --override-type=json --restart=OnFailure -- sleep 120 |
yq 'with(.spec.containers; .[1]=.[0] | .[0].name="alpine-1" | .[1].name="alpine-2")' |
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
    - "120"
    image: alpine:latest
    name: alpine-1
    volumeMounts:
    - mountPath: /cache
      name: alpine
  - args:
    - sleep
    - "120"
    image: alpine:latest
    name: alpine-2
    volumeMounts:
    - mountPath: /cache
      name: alpine
  restartPolicy: OnFailure
  volumes:
  - emptyDir:
      medium: Memory
      sizeLimit: 1Mi
    name: alpine

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   2/2     Running   0          7s

$ kubectl exec alpine --container=alpine-1 -- fallocate -l 256K /cache/alpine-1

$ kubectl exec alpine --container=alpine-2 -- fallocate -l 256K /cache/alpine-2

$ kubectl exec alpine --container=alpine-1 -- ls /cache
alpine-1
alpine-2

$ kubectl exec alpine --container=alpine-2 -- ls /cache
alpine-1
alpine-2

$ kubectl exec alpine --container=alpine-1 -- df -h /cache
Filesystem                Size      Used Available Use% Mounted on
tmpfs                     1.0M    512.0K    512.0K  50% /cache

$ kubectl exec alpine --container=alpine-2 -- df -h /cache
Filesystem                Size      Used Available Use% Mounted on
tmpfs                     1.0M    512.0K    512.0K  50% /cache

$ kubectl get pods --output=yaml | yq '.items | map({"name":.metadata.name,"node":.spec.nodeName})'
- name: alpine
  node: cluster-worker-yellow

$ docker exec cluster-worker-yellow mount | fgrep empty-dir
tmpfs on /var/lib/kubelet/pods/a1ecea4e-f795-4787-95be-ed2f40b5b9ac/volumes/kubernetes.io~empty-dir/alpine type
tmpfs (rw,relatime,size=1024k,inode64,noswap)

$ docker exec cluster-worker-yellow ls \
/var/lib/kubelet/pods/2d3a9737-3747-40ed-b4d7-bfb67fa94292/volumes/kubernetes.io~empty-dir/alpine
alpine-1
alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/2     Completed   0          2m7s

$ docker exec cluster-worker-yellow ls \
/var/lib/kubelet/pods/2d3a9737-3747-40ed-b4d7-bfb67fa94292/volumes/kubernetes.io~empty-dir

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ docker exec cluster-worker-yellow ls /var/lib/kubelet/pods/
f0338d9b-b37c-4537-888c-d2e5b83a6ebb
fdc41f67-343b-4c79-a89f-64fb035883a6

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
