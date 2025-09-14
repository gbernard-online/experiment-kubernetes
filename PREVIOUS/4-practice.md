[![Gmail](avatar.webp "ghislain.bernard@gmail.com")](mailto:ghislain.bernard@gmail.com) [![Github](github.webp "ghislain-bernard")](https://github.com/ghislain-bernard) [![LinkedIN](linkedin.webp "ghislain-bernard")](https://www.linkedin.com/in/ghislain-bernard)

## PRACTICE #19 - MICROK8S - UBUNTU 24

```bash
$ kubectl api-resources --no-headers | fgrep persistent
persistentvolumeclaims              pvc        v1                                true    PersistentVolumeClaim
persistentvolumes                   pv         v1                                false   PersistentVolume

$ kubectl explain persistentvolumes
KIND:       PersistentVolume
VERSION:    v1

DESCRIPTION:
    PersistentVolume (PV) is a storage resource provisioned by an administrator.
    It is analogous to a node. More info:
    https://kubernetes.io/docs/concepts/storage/persistent-volumes
...

$ kubectl explain persistentvolumeclaims
KIND:       PersistentVolumeClaim
VERSION:    v1

DESCRIPTION:
    PersistentVolumeClaim is a user's request for and claim to a persistent
    volume
...

$ echo 'apiVersion: v1
kind: PersistentVolume
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 3Gi
  nfs:
    path: /alpine/data
    server: ubuntu.qemu.lan' | kubectl-neat | tee persistentvolume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 3Gi
  nfs:
    path: /alpine/data
    server: ubuntu.qemu.lan

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

$ kubectl get persistentvolumes --output=wide
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE   VOLUMEMODE
alpine   3Gi        RWX            Retain           Available                          <unset>                          31s   Filesystem

$ echo 'apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi' | kubectl-neat | tee persistentvolumeclaim.yaml

$ kubeconform -verbose persistentvolumeclaim.yaml
persistentvolumeclaim.yaml - PersistentVolumeClaim alpine is valid

$ kubectl apply --filename=persistentvolumeclaim.yaml
persistentvolumeclaim/alpine created

$ kubectl get persistentvolumeclaims --output=wide
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
alpine   Bound    alpine   3Gi        RWX                           <unset>                 18s   Filesystem

$ kubectl get persistentvolumes
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
alpine   3Gi        RWX            Retain           Bound    default/alpine                  <unset>                          71s

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
alpine   1/1     Running   0          6s

$ kubectl exec alpine -- touch /data/alpine-1

$ kubectl exec alpine -- ls /data
alpine-1

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          68s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1

$ sudo touch /srv/nfs/alpine/data/alpine-2

$ sudo ls --width=1 /srv/nfs/alpine/data
alpine-1
alpine-2

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl exec alpine -- ls /data
alpine-1
alpine-2

$ kubectl exec alpine -- touch /data/alpine-3

$ kubectl exec alpine -- ls /data
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

$ kubectl delete --filename=persistentvolumeclaim.yaml
persistentvolumeclaim "alpine" deleted

$ kubectl delete --filename=persistentvolume.yaml
persistentvolume "alpine" deleted

$ sudo rm --recursive --verbose /srv/nfs/alpine
removed '/srv/nfs/alpine/data/alpine-3'
removed '/srv/nfs/alpine/data/alpine-2'
removed '/srv/nfs/alpine/data/alpine-1'
removed directory '/srv/nfs/alpine/data'
removed directory '/srv/nfs/alpine'

$ rm --verbose persistentvolume.yaml persistentvolumeclaim.yaml pod.yaml
removed 'persistentvolume.yaml'
removed 'persistentvolumeclaim.yaml'
removed 'pod.yaml'

```
