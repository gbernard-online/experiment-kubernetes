# DRAFT - EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset  
https://www.baeldung.com/ops/kubernetes-headless-service

https://www.youtube.com/watch?v=q7qEDZYxHzg&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #26 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain statefulset --output=plaintext-openapiv2

$ kubectl explain statefulset.spec --output=plaintext-openapiv2

$ kubectl explain statefulset.spec.ordinals --output=plaintext-openapiv2

$ kubectl explain statefulset.spec.volumeClaimTemplates --output=plaintext-openapiv2
KIND:     StatefulSet
VERSION:  apps/v1

RESOURCE: volumeClaimTemplates <[]Object>

DESCRIPTION:
     volumeClaimTemplates is a list of claims that pods are allowed to
     reference. The StatefulSet controller is responsible for mapping network
     identities to claims in a way that maintains the identity of a pod. Every
     claim in this list must have at least one matching (by name) volumeMount in
     one container in the template. A claim in this list takes precedence over
     any volumes in the template, with the same name.

     PersistentVolumeClaim is a user's request for and claim to a persistent
     volume

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     spec defines the desired characteristics of a volume requested by a pod
     author. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

   status	<Object>
     status represents the current information/status of a persistent volume
     claim. Read-only. More info:
     https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

$ kubectl create service clusterip nginx --clusterip=None --dry-run=client --output=yaml --tcp=80 |
kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  clusterIP: None
  ports:
  - name: "80"
    port: 80
  selector:
    app: nginx

$ kubectl apply --filename=service.yaml
service/nginx created

$ kubectl get endpointslices.discovery.k8s.io nginx-rxrbk 
NAME          ADDRESSTYPE   PORTS     ENDPOINTS   AGE
nginx-rxrbk   IPv4          <unset>   <unset>     15s

$ echo 'apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  ordinals:
    start: 1
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx
      initContainers:
      - command:
        - ash
        - '-c'
        - 'date | tee -a /usr/share/nginx/html/index.html'
        image: alpine:latest
        name: alpine
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx
  volumeClaimTemplates:
  - metadata:
      name: nginx
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi' | kubectl-neat | tee statefullset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx
  volumeClaimTemplates:
  - metadata:
      name: nginx
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

$ kubeconform -verbose statefullset.yaml 
statefullset.yaml - StatefulSet nginx is valid

$ kubectl apply --filename=statefullset.yaml

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- wget -O - -q -T 5 nginx-1.nginx.default.svc.cluster.local
Wed Oct  1 00:00:03 UTC 2025

user@ubuntu:[~/kubernetes]
$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- wget -O - -q -T 5 nginx-0.nginx.default.svc.cluster.local
Wed Oct  1 00:00:08 UTC 2025

user@ubuntu:[~/kubernetes]
$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- wget -O - -q -T 5 nginx-1.nginx.default.svc.cluster.local
Wed Oct  1 00:00:03 UTC 2025

$ kubectl get endpointslices.discovery.k8s.io nginx-rxrbk 
NAME          ADDRESSTYPE   PORTS   ENDPOINTS                           AGE
nginx-rxrbk   IPv4          80      10.244.1.8,10.244.2.11,10.244.3.8   37m

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  ordinals:
    start: 1
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx
      initContainers:
      - command:
        - ash
        - '-c'
        - 'date | tee -a /usr/share/nginx/html/index.html'
        image: alpine:latest
        name: alpine
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx
  volumeClaimTemplates:
  - metadata:
      name: nginx
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi


```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
