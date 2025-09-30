# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=UTfdoXmRevU&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #24 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain persistentvolumes.spec.hostPath --output=plaintext-openapiv2
|...|

$ kubectl explain persistentvolumes.spec.nodeAffinity --output=plaintext-openapiv2
|...|

$ kubectl explain persistentvolumes.spec.nodeAffinity.required.nodeSelectorTerms --output=plaintext-openapiv2
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

$ echo 'apiVersion: v1
kind: PersistentVolume
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /srv/kube/alpine/data
    type: DirectoryOrCreate
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: color
          operator: In
          values:
            - red
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
  hostPath:
    path: /srv/kube/alpine/data
    type: DirectoryOrCreate
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: color
          operator: In
          values:
          - red
  storageClassName: alpine

$ kubeconform -verbose persistentvolume.yaml
persistentvolume.yaml - PersistentVolume alpine is valid

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
alpine   1/1     Running   0          9s

$ kubectl get pods alpine --output=yaml | yq .spec.nodeName
cluster-worker-red

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .status.phase
Bound

$ kubectl get persistentvolume alpine --output=yaml | yq .status.phase
Bound

$ docker exec cluster-worker-red ls /srv/kube/alpine/data

$ kubectl exec alpine -- touch /data/alpine-1

$ kubectl exec alpine -- ls /data
alpine-1

$ docker exec cluster-worker-red ls /srv/kube/alpine/data
alpine-1

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          65s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ docker exec cluster-worker-red ls /srv/kube/alpine/data
alpine-1

$ docker exec cluster-worker-red touch /srv/kube/alpine/data/alpine-2

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          8s

$ kubectl get pods alpine --output=yaml | yq .spec.nodeName
cluster-worker-red

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
alpine   0/1     Completed   0          64s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ docker exec cluster-worker-red ls /srv/kube/alpine/data
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

$ docker exec cluster-worker-red rm --recursive --verbose /srv/kube
removed '/srv/kube/alpine/data/alpine-1'
removed '/srv/kube/alpine/data/alpine-3'
removed '/srv/kube/alpine/data/alpine-2'
removed directory '/srv/kube/alpine/data'
removed directory '/srv/kube/alpine'
removed directory '/srv/kube'

$ rm --verbose storageclass.yaml persistentvolume.yaml persistentvolumeclaim.yaml pod.yaml
removed 'storageclass.yaml'
removed 'persistentvolume.yaml'
removed 'persistentvolumeclaim.yaml'
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
