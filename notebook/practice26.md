# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset  
https://www.baeldung.com/ops/kubernetes-headless-service

https://www.youtube.com/watch?v=q7qEDZYxHzg&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #26 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep statefulsets
statefulsets                        sts        apps/v1                           true    StatefulSet

$ kubectl explain statefulsets --output=plaintext-openapiv2
|...|

$ kubectl explain statefulsets.spec --output=plaintext-openapiv2
|...|

$ kubectl explain statefulsets.spec.ordinals --output=plaintext-openapiv2
|...|

$ kubectl explain statefulsets.spec.volumeClaimTemplates --output=plaintext-openapiv2
|...|

$ kubectl explain statefulsets.spec.volumeClaimTemplates.spec --output=plaintext-openapiv2
|...|

$ kubectl explain statefulsets.spec.volumeClaimTemplates.spec --output=plaintext-openapiv2 --recursive
|...|
```

```bash
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

$ kubectl get services nginx
NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   None         <none>        80/TCP    10s

$ kubectl get endpointslices.discovery.k8s.io nginx-x4x2f
NAME          ADDRESSTYPE   PORTS     ENDPOINTS   AGE
nginx-x4x2f   IPv4          <unset>   <unset>     15s

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
        - -c
        - sleep 60 && echo $KUBERNETES_CONTAINER_NAME | tee /usr/share/nginx/html/index.html
        env:
          - name: KUBERNETES_CONTAINER_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
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
        - -c
        - sleep 60 && echo $KUBERNETES_CONTAINER_NAME | tee /usr/share/nginx/html/index.html
        env:
        - name: KUBERNETES_CONTAINER_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
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

$ kubeconform -verbose statefullset.yaml
statefullset.yaml - StatefulSet nginx is valid

$ kubectl apply --filename=statefullset.yaml
statefulset.apps/nginx created

$ kubectl annotate statefulsets.apps nginx kubernetes.io/change-cause=nginx:alpine
statefulset.apps/nginx annotated

$ kubectl rollout history statefulset nginx
statefulset.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>

$ kubectl get statefulsets.apps nginx
NAME    READY   AGE
nginx   0/3     31s

$ kubectl get pods --selector=app=nginx
NAME      READY   STATUS     RESTARTS   AGE
nginx-1   0/1     Init:0/1   0          45s

$ kubectl get statefulsets.apps nginx
NAME    READY   AGE
nginx   1/3     74s

$ kubectl get pods --selector=app=nginx
NAME      READY   STATUS     RESTARTS   AGE
nginx-1   1/1     Running    0          2m15s
nginx-2   0/1     Init:0/1   0          66s

$ kubectl get statefulsets.apps nginx
NAME    READY   AGE
nginx   3/3     3m18s

$ kubectl get pods --selector=app=nginx
NAME      READY   STATUS    RESTARTS   AGE
nginx-1   1/1     Running   0          3m28s
nginx-2   1/1     Running   0          2m19s
nginx-3   1/1     Running   0          72s

$ kubectl get endpointslices.discovery.k8s.io nginx-x4x2f
NAME          ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
nginx-x4x2f   IPv4          80      10.244.1.14,10.244.3.13,10.244.2.14   4m

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-yellow
cluster-worker-green

$ kubectl get persistentvolumeclaims --output=name --selector=app=nginx
persistentvolumeclaim/nginx-nginx-1
persistentvolumeclaim/nginx-nginx-2
persistentvolumeclaim/nginx-nginx-3

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx-1.nginx.default.svc.cluster.local
nginx-1

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx-2.nginx.default.svc.cluster.local
nginx-2

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx-3.nginx.default.svc.cluster.local
nginx-3

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- nslookup nginx.default.svc.cluster.local | cat --squeeze-blank
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	nginx.default.svc.cluster.local
Address: 10.244.1.14
Name:	nginx.default.svc.cluster.local
Address: 10.244.3.13
Name:	nginx.default.svc.cluster.local
Address: 10.244.2.14

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx.default.svc.cluster.local
nginx-1

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- wget -O - -q -T 5 nginx.default.svc.cluster.local
nginx-2

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- wget -O - -q -T 5 nginx.default.svc.cluster.local
nginx-3

$ kubectl scale statefulset nginx --replicas=6
statefulset.apps/nginx scaled

$ kubectl get statefulsets.apps nginx
NAME    READY   AGE
nginx   3/6     8m

$ kubectl get statefulsets.apps nginx
NAME    READY   AGE
nginx   4/6     9m

$ kubectl get statefulsets.apps nginx
NAME    READY   AGE
nginx   5/6     10m

$ kubectl get statefulsets.apps nginx
NAME    READY   AGE
nginx   5/6     11m

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-yellow
cluster-worker-green
cluster-worker-green
cluster-worker-red
cluster-worker-yellow

$ kubectl get persistentvolumeclaims --output=name --selector=app=nginx
persistentvolumeclaim/nginx-nginx-1
persistentvolumeclaim/nginx-nginx-2
persistentvolumeclaim/nginx-nginx-3
persistentvolumeclaim/nginx-nginx-4
persistentvolumeclaim/nginx-nginx-5
persistentvolumeclaim/nginx-nginx-6

$ kubectl delete --filename=service.yaml
service "nginx" deleted

$ kubectl delete --filename=statefullset.yaml
statefulset.apps "nginx" deleted

$ kubectl delete persistentvolumeclaims --selector=app=nginx
persistentvolumeclaim "nginx-nginx-1" deleted
persistentvolumeclaim "nginx-nginx-2" deleted
persistentvolumeclaim "nginx-nginx-3" deleted
persistentvolumeclaim "nginx-nginx-4" deleted
persistentvolumeclaim "nginx-nginx-5" deleted
persistentvolumeclaim "nginx-nginx-6" deleted

$ rm --verbose service.yaml statefullset.yaml
removed 'service.yaml'
removed 'statefullset.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
