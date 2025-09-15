# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/storage/persistent-volumes
https://kubernetes.io/docs/concepts/storage/storage-classes

https://www.youtube.com/watch?v=cfKwoAkY3F8&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=1HrHv_HBxwI&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #16 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep storageclass
storageclasses                      sc         storage.k8s.io/v1                 false   StorageClass

$ kubectl api-resources --no-headers | fgrep persistentvolumes
persistentvolumes                   pv         v1                                false   PersistentVolume

$ kubectl api-resources --no-headers | fgrep persistentvolumeclaims
persistentvolumeclaims              pvc        v1                                true    PersistentVolumeClaim

$ kubectl explain kubectl explain storageclasses
GROUP:      storage.k8s.io
KIND:       StorageClass
VERSION:    v1

DESCRIPTION:
    StorageClass describes the parameters for a class of storage for which
    PersistentVolumes can be dynamically provisioned.

    StorageClasses are non-namespaced; the name of the storage class according
    to etcd is in ObjectMeta.Name.
|...|

$ kubectl explain persistentvolumes
KIND:       PersistentVolume
VERSION:    v1

DESCRIPTION:
    PersistentVolume (PV) is a storage resource provisioned by an administrator.
    It is analogous to a node. More info:
    https://kubernetes.io/docs/concepts/storage/persistent-volumes
|...|

$ kubectl explain persistentvolumeclaims
KIND:       PersistentVolumeClaim
VERSION:    v1

DESCRIPTION:
    PersistentVolumeClaim is a userʼs request for and claim to a persistent
    volume
|...|
```

```bash
$ echo 'apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: alpine
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer' | kubectl-neat | tee storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: alpine
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer

$ kubeconform -verbose storageclass.yaml
storageclass.yaml - StorageClass alpine is valid

$ kubectl apply --filename=storageclass.yaml
storageclass.yaml - StorageClass alpine created.yaml

$ kubectl get storageclasses alpine --output=yaml | yq .reclaimPolicy,.volumeBindingMode
Retain
WaitForFirstConsumer

$ echo 'apiVersion: v1
kind: PersistentVolume
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  nfs:
    path: /alpine/data
    server: ubuntu.qemu.lan
  storageClassName: alpine' | kubectl-neat | tee persistentvolume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  nfs:
    path: /alpine/data
    server: ubuntu.qemu.lan
  storageClassName: alpine

$ kubeconform -verbose persistentvolume.yaml
persistentvolume.yaml - PersistentVolume alpine is valid

$ sudo mkdir --parents --verbose /srv/nfs/alpine/data
mkdir: created directory '/srv/nfs/alpine'
mkdir: created directory '/srv/nfs/alpine/data'

$ sudo chmod --recursive --verbose g-rx,o-rx /srv/nfs/alpine
mode of '/srv/nfs/alpine' changed from 0755 (rwxr-xr-x) to 0700 (rwx------)
mode of '/srv/nfs/alpine/data' changed from 0755 (rwxr-xr-x) to 0700 (rwx------)

$ kubectl apply --filename=persistentvolume.yaml
persistentvolume/alpine created

$ kubectl get persistentvolumes alpine --output=yaml | yq .status.phase
Available

$ echo 'apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: alpine' | kubectl-neat | tee persistentvolumeclaim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: alpine

$ kubeconform -verbose persistentvolumeclaim.yaml
persistentvolumeclaim.yaml - PersistentVolumeClaim alpine is valid

$ kubectl apply --filename=persistentvolumeclaim.yaml
persistentvolumeclaim/alpine created

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .status.phase
Pending

$ kubectl get persistentvolumes alpine --output=yaml | yq .status.phase
Available

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
      "name": "alpine",
      "persistentVolumeClaim":
      {
        "claimName": "alpine"
      }
    }
  ]
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
    volumeMounts:
    - mountPath: /data
      name: alpine
  restartPolicy: OnFailure
  volumes:
  - name: alpine
    persistentVolumeClaim:
      claimName: alpine

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          11s

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .status.phase
Bound

$ kubectl get persistentvolume alpine --output=yaml | yq .status.phase
Bound

$ kubectl exec alpine -- touch /data/alpine-1

$ kubectl exec alpine -- ls /data
alpine-1

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          68s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1

$ sudo touch /srv/nfs/alpine/data/alpine-2

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl exec alpine -- ls /data
alpine-1
alpine-2

$ kubectl exec alpine -- touch /data/alpine-3

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1
alpine-2
alpine-3

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          67s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1
alpine-2
alpine-3

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .status.phase
Bound

$ kubectl get persistentvolume alpine --output=yaml | yq .status.phase
Bound

$ kubectl delete --filename=persistentvolumeclaim.yaml
persistentvolumeclaim "alpine" deleted

$ kubectl get persistentvolume alpine --output=yaml | yq .status.phase
Released

$ kubectl delete --filename=persistentvolume.yaml
persistentvolume "alpine" deleted

$ kubectl delete --filename=storageclass.yaml
storageclass.storage.k8s.io "alpine" deleted

$ sudo rm --recursive --verbose /srv/nfs/alpine
removed '/srv/nfs/alpine/data/alpine-3'
removed '/srv/nfs/alpine/data/alpine-2'
removed '/srv/nfs/alpine/data/alpine-1'
removed directory '/srv/nfs/alpine/data'
removed directory '/srv/nfs/alpine'

$ rm --verbose storageclass.yaml persistentvolume.yaml persistentvolumeclaim.yaml pod.yaml
removed 'storageclass.yaml'
removed 'persistentvolume.yaml'
removed 'persistentvolumeclaim.yaml'
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
