# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=kfk9deenCpE&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #15 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.volumes.nfs --output=plaintext-openapiv2
|...|

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/volumeMounts",
  "value": [
    {
      "mountPath": "/data",
      "name": "alpine",
      "subPath": "alpine/data"
    }
  ]
},
{
  "op": "add",
  "path": "/spec/volumes",
  "value": [
    {
      "name": "alpine",
      "nfs": {
        "path": "/",
        "server": "ubuntu.qemu.lan"
      }
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
    - mountPath: /data
      name: alpine
      subPath: alpine/data
  - args:
    - sleep
    - "120"
    image: alpine:latest
    name: alpine-2
    volumeMounts:
    - mountPath: /data
      name: alpine
      subPath: alpine/data
  restartPolicy: OnFailure
  volumes:
  - name: alpine
    nfs:
      path: /
      server: ubuntu.qemu.lan

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   2/2     Running   0          5s

$ sudo ls --width=1 /srv/nfs/alpine/data

$ kubectl exec alpine --container=alpine-1 -- touch /data/alpine-1

$ kubectl exec alpine --container=alpine-2 -- touch /data/alpine-2

$ kubectl exec alpine --container=alpine-1 -- ls /data
alpine-1
alpine-2

$ kubectl exec alpine --container=alpine-2 -- ls /data
alpine-1
alpine-2

$ kubectl exec alpine --container=alpine-1 -- df -h /data
Filesystem                Size      Used Available Use% Mounted on
ubuntu.qemu.lan:/alpine/data
                         30.3G     13.3G     15.5G  46% /data

kubectl exec alpine --container=alpine-2 -- df -h /data
Filesystem                Size      Used Available Use% Mounted on
ubuntu.qemu.lan:/alpine/data
                         30.3G     13.3G     15.5G  46% /data

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1
alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/2     Completed   0          2m20s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1
alpine-2

$ sudo rm --recursive --verbose /srv/nfs/alpine
removed '/srv/nfs/alpine/data/alpine-1'
removed '/srv/nfs/alpine/data/alpine-2'
removed directory '/srv/nfs/alpine/data'
removed directory '/srv/nfs/alpine'

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
