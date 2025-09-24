# WIP: EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=Mz5d8Miphso&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #17 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl get storageclasses --output=name
storageclass.storage.k8s.io/microk8s-hostpath

$ kubectl get storageclasses microk8s-hostpath --output=yaml | yq .reclaimPolicy,.volumeBindingMode
Delete
WaitForFirstConsumer

$ echo 'apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi' | kubectl-neat | tee persistentvolumeclaim.yaml
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

$ kubeconform -verbose persistentvolumeclaim.yaml
persistentvolumeclaim.yaml - PersistentVolumeClaim alpine is valid

$ kubectl apply --filename=persistentvolumeclaim.yaml
persistentvolumeclaim/alpine created

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .spec.storageClassName,.status.phase
microk8s-hostpath
Pending

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

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .status.phase,.spec.volumeName
Bound
pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e

$ kubectl exec alpine -- touch /data/alpine-1

$ kubectl exec alpine -- ls /data
alpine-1

$ fd --base-directory=/var/snap/microk8s/common/default-storage alpine-
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-1

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          83s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ fd --base-directory=/var/snap/microk8s/common/default-storage alpine-
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-1

$ (cd /var/snap/microk8s/common/default-storage && \
sudo touch default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-2)

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl exec alpine -- ls /data
alpine-1
alpine-2

$ kubectl exec alpine -- touch /data/alpine-3

$ fd --base-directory=/var/snap/microk8s/common/default-storage alpine-
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-1
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-2
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-3

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          66s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ fd --base-directory=/var/snap/microk8s/common/default-storage alpine-
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-1
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-2
default-alpine-pvc-52603bc4-53f4-442b-96bb-c86c2cd4505e/alpine-3

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .status.phase
Bound

$ kubectl delete --filename=persistentvolumeclaim.yaml
persistentvolumeclaim "alpine" deleted

$ fd --base-directory=/var/snap/microk8s/common/default-storage alpine-

$ rm --verbose persistentvolumeclaim.yaml pod.yaml
removed 'persistentvolumeclaim.yaml'
removed 'pod.yaml'
```

## PRACTICE #16 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl get storageclasses --output=name
storageclass.storage.k8s.io/standard

$ kubectl get storageclasses standard --output=yaml | yq .reclaimPolicy,.volumeBindingMode
Delete
WaitForFirstConsumer

$ echo 'apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpine
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi' | kubectl-neat | tee persistentvolumeclaim.yaml
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

$ kubeconform -verbose persistentvolumeclaim.yaml
persistentvolumeclaim.yaml - PersistentVolumeClaim alpine is valid

$ kubectl apply --filename=persistentvolumeclaim.yaml
persistentvolumeclaim/alpine created

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .spec.storageClassName,.status.phase
standard
Pending

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

$ kubectl get persistentvolumeclaims alpine --output=yaml | 
yq '.status.phase,.spec.volumeName,.metadata.annotations."volume.kubernetes.io/selected-node"'
Bound
pvc-e57011af-591c-4501-ac13-64c93172ecbe
cluster-worker-yellow

$ kubectl get pods alpine --output=yaml | yq .spec.nodeName
cluster-worker-yellow

$ kubectl exec alpine -- touch /data/alpine-1

$ kubectl exec alpine -- ls /data
alpine-1

$ docker exec cluster-worker-yellow \
find /var/local-path-provisioner -name 'alpine-*' -printf '%P\n'
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-1

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          83s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ docker exec cluster-worker-yellow \
find /var/local-path-provisioner -name 'alpine-*' -printf '%P\n'
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-1

$ docker exec cluster-worker-yellow \
touch /var/local-path-provisioner/pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-2

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine --output=yaml | yq .spec.nodeName
cluster-worker-yellow

$ kubectl exec alpine -- ls /data
alpine-1
alpine-2

$ kubectl exec alpine -- touch /data/alpine-3

$ docker exec cluster-worker-yellow \
find /var/local-path-provisioner -name 'alpine-*' -printf '%P\n'
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-3
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-1
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          66s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ docker exec cluster-worker-yellow \
find /var/local-path-provisioner -name 'alpine-*' -printf '%P\n'
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-3
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-1
pvc-ef852a11-0e25-4431-99e9-77ccf8ee98f6_default_alpine/alpine-2

$ kubectl get persistentvolumeclaims alpine --output=yaml | yq .status.phase
Bound

$ kubectl delete --filename=persistentvolumeclaim.yaml
persistentvolumeclaim "alpine" deleted

$ docker exec cluster-worker-yellow \
find /var/local-path-provisioner -name 'alpine-*' -printf '%P\n'

$ rm --verbose persistentvolumeclaim.yaml pod.yaml
removed 'persistentvolumeclaim.yaml'
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
