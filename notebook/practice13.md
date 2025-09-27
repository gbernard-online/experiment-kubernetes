# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/storage/volumes

https://www.youtube.com/watch?v=0KSOqB4nea0&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #13 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.volumes.hostPath --output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: hostPath <Object>

DESCRIPTION:
     hostPath represents a pre-existing file or directory on the host machine
     that is directly exposed to the container. This is generally used for
     system agents or other privileged things that are allowed to see the host
     machine. Most containers will NOT need this. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#hostpath

     Represents a host path mapped into a pod. Host path volumes do not support
     ownership management or SELinux relabeling.

FIELDS:
   path	<string> -required-
     path of the directory on the host. If the path is a symlink, it will follow
     the link to the real path. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#hostpath

   type	<string>
     type for HostPath Volume Defaults to "" More info:
     https://kubernetes.io/docs/concepts/storage/volumes#hostpath

     Possible enum values:
     - `""` For backwards compatible, leave it empty if unset
     - `"BlockDevice"` A block device must exist at the given path
     - `"CharDevice"` A character device must exist at the given path
     - `"Directory"` A directory must exist at the given path
     - `"DirectoryOrCreate"` If nothing exists at the given path, an empty
     directory will be created there as needed with file mode 0755, having the
     same group and ownership with Kubelet.
     - `"File"` A file must exist at the given path
     - `"FileOrCreate"` If nothing exists at the given path, an empty file will
     be created there as needed with file mode 0644, having the same group and
     ownership with Kubelet.
     - `"Socket"` A UNIX socket must exist at the given path

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/volumeMounts",
  "value": [
    {
      "mountPath": "/data",
      "name": "alpine"
    }
  ]
},
{
  "op": "add",
  "path": "/spec/volumes",
  "value": [
    {
      "hostPath":
      {
        "path": "/srv/kube/alpine/data",
        "type": "DirectoryOrCreate"
      },
      "name": "alpine"
    }
  ]
}
]' --override-type=json --restart=OnFailure -- sleep 120 | kubectl-neat | tee pod.yaml
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
    volumeMounts:
    - mountPath: /data
      name: alpine
  restartPolicy: OnFailure
  volumes:
  - hostPath:
      path: /srv/kube/alpine/data
      type: DirectoryOrCreate
    name: alpine

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          8s

$ kubectl get pods --output=yaml |
yq '.items | map({"name":.metadata.name,"node":.spec.nodeName})'
- name: alpine
  node: cluster-worker-red

$ docker exec cluster-worker-red ls /srv/kube/alpine/data

$ kubectl exec alpine -- touch /data/alpine-1

$ kubectl exec alpine -- ls /data
alpine-1

$ docker exec cluster-worker-red ls /srv/kube/alpine/data
alpine-1

$ docker exec cluster-worker-red touch /srv/kube/alpine/data/alpine-2

$ docker exec cluster-worker-red ls /srv/kube/alpine/data
alpine-1
alpine-2

$ kubectl exec alpine -- ls /data
alpine-1
alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          2m24s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ docker exec cluster-worker-red ls /srv/kube/alpine/data
alpine-1
alpine-2

$ docker exec cluster-worker-red rm --recursive --verbose /srv/kube
removed '/srv/kube/alpine/data/alpine-1'
removed '/srv/kube/alpine/data/alpine-2'
removed directory '/srv/kube/alpine/data'
removed directory '/srv/kube/alpine'
removed directory '/srv/kube'

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
