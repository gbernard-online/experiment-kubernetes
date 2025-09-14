

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
]' --override-type=json --restart=OnFailure -- sleep 60 | yq 'with(.spec.containers; .[1]=.[0] | (.[0].name,.[0].env[0].value)="alpine-1" | (.[1].name,.[1].env[0].value)="alpine-2")' | kubectl-neat | tee pod.yaml
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

## PRACTICE #13 - MICROK8S - UBUNTU 24

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
]' --override-type=json --restart=OnFailure -- sleep 120 | yq 'with(.spec.containers; .[1]=.[0] | .[0].name="alpine-1" | .[1].name="alpine-2")' | kubectl-neat | tee pod.yaml
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
/dev/vda2                14.9G     11.7G      2.5G  83% /cache

$ kubectl exec alpine --container=alpine-2 -- df -h /cache
Filesystem                Size      Used Available Use% Mounted on
/dev/vda2                14.9G     11.7G      2.5G  83% /cache

$ sudo fdfind alpine- /var/snap/microk8s/common/var/lib/kubelet/pods
/var/snap/microk8s/common/var/lib/kubelet/pods/82c68e53-4347-40d4-86c7-6ed2350d340b/containers/alpine-1/
/var/snap/microk8s/common/var/lib/kubelet/pods/82c68e53-4347-40d4-86c7-6ed2350d340b/containers/alpine-2/
/var/snap/microk8s/common/var/lib/kubelet/pods/82c68e53-4347-40d4-86c7-6ed2350d340b/volumes/kubernetes.io~empty-dir/alpine/alpine-1
/var/snap/microk8s/common/var/lib/kubelet/pods/82c68e53-4347-40d4-86c7-6ed2350d340b/volumes/kubernetes.io~empty-dir/alpine/alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/2     Completed   0          2m6s

$ sudo fdfind alpine- /var/snap/microk8s/common/var/lib/kubelet/pods
/var/snap/microk8s/common/var/lib/kubelet/pods/82c68e53-4347-40d4-86c7-6ed2350d340b/containers/alpine-1/
/var/snap/microk8s/common/var/lib/kubelet/pods/82c68e53-4347-40d4-86c7-6ed2350d340b/containers/alpine-2/

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ sudo fdfind alpine- /var/snap/microk8s/common/var/lib/kubelet/pods

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
]' --override-type=json --restart=OnFailure -- sleep 120 | yq 'with(.spec.containers; .[1]=.[0] | .[0].name="alpine-1" | .[1].name="alpine-2")' | kubectl-neat | tee pod.yaml
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

$ mount | fgrep empty-dir
tmpfs on /var/snap/microk8s/common/var/lib/kubelet/pods/8c7ba922-9bbf-436a-9d3a-b1483f7079a4/volumes/kubernetes.io~empty-dir/alpine type tmpfs (rw,relatime,size=1024k,inode64,noswap)

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

$ sudo ls /var/snap/microk8s/common/var/lib/kubelet/pods/8c7ba922-9bbf-436a-9d3a-b1483f7079a4/volumes/kubernetes.io~empty-dir/alpine
alpine-1  alpine-2

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/2     Completed   0          2m7s

$ sudo ls /var/snap/microk8s/common/var/lib/kubelet/pods/8c7ba922-9bbf-436a-9d3a-b1483f7079a4/volumes/kubernetes.io~empty-dir/alpine
ls: cannot access '/var/snap/microk8s/common/var/lib/kubelet/pods/8c7ba922-9bbf-436a-9d3a-b1483f7079a4/volumes/kubernetes.io~empty-dir/alpine': No such file or directory

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'

```

## PRACTICE #14 - MICROK8S - UBUNTU 24

```bash
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
  - hostPath:
      path: /srv/kube/alpine/data
      type: DirectoryOrCreate
    name: alpine

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          8s

$ kubectl exec alpine -- touch /data/alpine-1

$ kubectl exec alpine -- ls /data
alpine-1

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          68s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ ls --width=1 /srv/kube/alpine/data
alpine-1

$ sudo touch /srv/kube/alpine/data/alpine-2

$ ls --width=1 /srv/kube/alpine/data
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

$ ls --width=1 /srv/kube/alpine/data
alpine-1
alpine-2
alpine-3

$ sudo rm --recursive --verbose /srv/kube
removed '/srv/kube/alpine/data/alpine-3'
removed '/srv/kube/alpine/data/alpine-2'
removed '/srv/kube/alpine/data/alpine-1'
removed directory '/srv/kube/alpine/data'
removed directory '/srv/kube/alpine'
removed directory '/srv/kube'

$ rm --verbose pod.yaml
removed 'pod.yaml'

```

## PRACTICE #15 - MICROK8S - UBUNTU 24

```bash
$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
ubuntu   Ready    <none>   11d   v1.32.3

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/nodeSelector",
  "value": {
    "kubernetes.io/hostname": "ubuntu"
  }
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
  nodeSelector:
    kubernetes.io/hostname: ubuntu
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeSelector}' && echo
{"kubernetes.io/hostname":"ubuntu"}

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeName}' && echo
ubuntu

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          67s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'

```

## PRACTICE #16 - MICROK8S - UBUNTU 24

```bash
$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
ubuntu   Ready    <none>   11d   v1.32.3

$ kubectl get nodes --output=jsonpath-as-json='{.items[*].metadata.labels}'
[
    {
        "beta.kubernetes.io/arch": "amd64",
        "beta.kubernetes.io/os": "linux",
        "kubernetes.io/arch": "amd64",
        "kubernetes.io/hostname": "ubuntu",
        "kubernetes.io/os": "linux",
        "microk8s.io/cluster": "true",
        "node.kubernetes.io/microk8s-controlplane": "microk8s-controlplane"
    }
]

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/nodeSelector",
  "value": {
    "kubernetes.io/arch": "amd64",
    "kubernetes.io/os": "linux"
  }
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
  nodeSelector:
    kubernetes.io/arch: amd64
    kubernetes.io/os: linux
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeSelector}' && echo
{"kubernetes.io/arch":"amd64","kubernetes.io/os":"linux"}

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeName}' && echo
ubuntu

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          67s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'

```

## PRACTICE #17 - KIND - CLUSTER-COLOR - UBUNTU 24

```bash
$ kubectl get nodes
NAME                          STATUS   ROLES           AGE     VERSION
cluster-color-control-plane   Ready    control-plane   8m15s   v1.33.1
cluster-color-worker-green    Ready    <none>          8m2s    v1.33.1
cluster-color-worker-orange   Ready    <none>          8m3s    v1.33.1
cluster-color-worker-red      Ready    <none>          8m2s    v1.33.1

$ kubectl get nodes --output=jsonpath-as-json='{.items[*].metadata.labels}' --selector=color
[
    {
        "beta.kubernetes.io/arch": "amd64",
        "beta.kubernetes.io/os": "linux",
        "color": "green",
        "kubernetes.io/arch": "amd64",
        "kubernetes.io/hostname": "cluster-color-worker-green",
        "kubernetes.io/os": "linux"
    },
    {
        "beta.kubernetes.io/arch": "amd64",
        "beta.kubernetes.io/os": "linux",
        "color": "orange",
        "kubernetes.io/arch": "amd64",
        "kubernetes.io/hostname": "cluster-color-worker-orange",
        "kubernetes.io/os": "linux"
    },
    {
        "beta.kubernetes.io/arch": "amd64",
        "beta.kubernetes.io/os": "linux",
        "color": "red",
        "kubernetes.io/arch": "amd64",
        "kubernetes.io/hostname": "cluster-color-worker-red",
        "kubernetes.io/os": "linux"
    }
]

```

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --patch='[
{
  "op": "add",
  "path": "/spec/nodeSelector",
  "value": {
    "color": "red"
  }
}
]' --type=json --restart=OnFailure -- sleep 60 | kubectl-neat | tee pod.yaml
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
  nodeSelector:
    color: red
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine --output=wide
NAME     READY   STATUS    RESTARTS   AGE   IP           NODE                       NOMINATED NODE   READINESS GATES
alpine   1/1     Running   0          13s   10.244.4.4   cluster-color-worker-red   <none>           <none>

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeSelector}' && echo
{"color":"red"}

$ kubectl get pods alpine --output=jsonpath='{.spec.nodeName}' && echo
cluster-color-worker-red

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          86s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'

```

```bash
$ kubectl create deployment nginx --dry-run=client --output=yaml --image=nginx:alpine --replicas=3 | kubectl-neat | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl get pods --output=wide --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE                          NOMINATED NODE   READINESS GATES
nginx-6b66fbbd46-hdhpt   1/1     Running   0          59s   10.244.3.49   cluster-color-worker-red      <none>           <none>
nginx-6b66fbbd46-msfnn   1/1     Running   0          59s   10.244.2.18   cluster-color-worker-orange   <none>           <none>
nginx-6b66fbbd46-xfc2d   1/1     Running   0          59s   10.244.1.16   cluster-color-worker-green    <none>           <none>

$ kubectl scale deployment nginx --replicas=9
deployment.apps/nginx scaled

$ kubectl get pods --output=wide --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE                          NOMINATED NODE   READINESS GATES
nginx-6b66fbbd46-2mpdk   1/1     Running   0          6s    10.244.3.10   cluster-color-worker-red      <none>           <none>
nginx-6b66fbbd46-d87r9   1/1     Running   0          6s    10.244.1.8    cluster-color-worker-green    <none>           <none>
nginx-6b66fbbd46-f58mn   1/1     Running   0          6s    10.244.1.7    cluster-color-worker-green    <none>           <none>
nginx-6b66fbbd46-s4ksz   1/1     Running   0          39s   10.244.3.9    cluster-color-worker-red      <none>           <none>
nginx-6b66fbbd46-sqpcg   1/1     Running   0          39s   10.244.2.6    cluster-color-worker-orange   <none>           <none>
nginx-6b66fbbd46-sxcqc   1/1     Running   0          6s    10.244.2.7    cluster-color-worker-orange   <none>           <none>
nginx-6b66fbbd46-x6slc   1/1     Running   0          39s   10.244.1.6    cluster-color-worker-green    <none>           <none>
nginx-6b66fbbd46-xdrzj   1/1     Running   0          6s    10.244.2.8    cluster-color-worker-orange   <none>           <none>
nginx-6b66fbbd46-xn6j5   1/1     Running   0          6s    10.244.3.11   cluster-color-worker-red      <none>           <none>

$ kubectl patch deployments.apps nginx --patch='[
{
  "op": "add",
  "path": "/spec/template/spec/nodeSelector",
  "value": {
    "color": "red"
  }
}
]' --type=json
deployment.apps/nginx patched

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine:red
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine
2         nginx:alpine:red

$ kubectl get pods --output=wide --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE                       NOMINATED NODE   READINESS GATES
nginx-7d9b6d6487-bjgz4   1/1     Running   0          39s   10.244.3.52   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-dhlb7   1/1     Running   0          39s   10.244.3.53   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-gb57p   1/1     Running   0          39s   10.244.3.54   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-kwb8p   1/1     Running   0          37s   10.244.3.57   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-l7sdm   1/1     Running   0          39s   10.244.3.56   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-lqzrv   1/1     Running   0          36s   10.244.3.60   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-m5dd5   1/1     Running   0          37s   10.244.3.58   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-snxbc   1/1     Running   0          36s   10.244.3.59   cluster-color-worker-red   <none>           <none>
nginx-7d9b6d6487-vpt8t   1/1     Running   0          39s   10.244.3.55   cluster-color-worker-red   <none>           <none>

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'

```

## PRACTICE #18 - MICROK8S - UBUNTU 24

```bash
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
]' --override-type=json --restart=OnFailure -- sleep 120 | yq 'with(.spec.containers; .[1]=.[0] | .[0].name="alpine-1" | .[1].name="alpine-2")' | kubectl-neat | tee pod.yaml
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

$ kubectl exec alpine --container=alpine-1 -- ls /data

$ kubectl exec alpine --container=alpine-2 -- ls /data

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
                         14.9G     10.4G      3.7G  74% /data

kubectl exec alpine --container=alpine-2 -- df -h /data
Filesystem                Size      Used Available Use% Mounted on
ubuntu.qemu.lan:/alpine/data
                         14.9G     10.4G      3.7G  74% /data

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
