[![Gmail](avatar.webp "ghislain.bernard@gmail.com")](mailto:ghislain.bernard@gmail.com) [![Github](github.webp "ghislain-bernard")](https://github.com/ghislain-bernard) [![LinkedIN](linkedin.webp "ghislain-bernard")](https://www.linkedin.com/in/ghislain-bernard)

## PRACTICE #1 - MICROK8S - UBUNTU 24

[![Kubernetes](kubernetes.webp "Kubernetes")](https://kubernetes.io)

```bash
$ kubectl get pods
No resources found in default namespace.

$ kubectl run nginx --image=nginx:alpine
pod/nginx created

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          5m30s

$ kubectl get pods --output=wide
NAME    READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          5m34s   10.1.243.197   ubuntu   <none>           <none>

$ kubectl get pods --no-headers --output=wide
nginx   1/1   Running   0     5m42s   10.1.243.197   ubuntu   <none>   <none>

$ curl --fail --show-error --silent http://10.1.243.197
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ kubectl get pods nginx --output=yaml
apiVersion: v1
kind: Pod
metadata:
...

$ kubectl describe pods nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             ubuntu/172.16.0.225
...

$ kubectl delete pods nginx
pod "nginx" deleted

```

```bash
$ kubectl run alpine --image=alpine:latest --overrides='{"spec":{"restartPolicy":"OnFailure"}}' -- sleep 10
pod/alpine created

$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          3s

$ kubectl get pods
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          14s

$ kubectl delete pods alpine
pod "alpine" deleted

```

```bash
$ kubectl run alpine --image=alpine:latest --rm --stdin --tty -- ash
If you don't see a command prompt, try pressing enter.
/ # apk add --no-cache --no-progress --quiet curl
...

```

## PRACTICE #2 - MICROK8S - UBUNTU 24

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure  -- sleep 10
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - sleep
    - "10"
    image: alpine:latest
    name: alpine
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: OnFailure
status: {}

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure -- sleep 10 | kubectl-neat | tee pod.yaml
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
    - "10"
    image: alpine:latest
    name: alpine
  restartPolicy: OnFailure

$ kubectl explain pods
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
...

$ kubectl explain pods.spec --recursive
KIND:       Pod
VERSION:    v1

FIELD: spec <PodSpec>


DESCRIPTION:
...

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl apply --filename=pod.yaml
pod/alpine unchanged

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          18s

$ kubectl get pods alpine --output=yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: db21901fc8440d95e170fe716b6d0e84346c9b8ad995478ffacc34dac5296350
    cni.projectcalico.org/podIP: ""
    cni.projectcalico.org/podIPs: ""
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"run":"alpine"},"name":"alpine","namespace":"default"},"spec":{"containers":[{"args":["sleep","10"],"image":"alpine:latest","name":"alpine"}],"restartPolicy":"OnFailure"}}
  creationTimestamp: "2025-07-09T16:30:33Z"
  labels:
    run: alpine
  name: alpine
  namespace: default
  resourceVersion: "8568"
  uid: c77045bd-4661-4ddd-be0e-076ee61c574b
...

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'

```

## PRACTICE #3 - MICROK8S - UBUNTU 24

```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=yaml | kubectl-neat
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx

$ kubectl run nginx --image=nginx:alpine
nginx

$ kubectl logs pods/nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/07/09 20:34:57 [notice] 1#1: using the "epoll" event method
2025/07/09 20:34:57 [notice] 1#1: nginx/1.29.0
2025/07/09 20:34:57 [notice] 1#1: built by gcc 14.2.0 (Alpine 14.2.0)
2025/07/09 20:34:57 [notice] 1#1: OS: Linux 5.15.0-143-generic
2025/07/09 20:34:57 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 65536:65536
2025/07/09 20:34:57 [notice] 1#1: start worker processes
2025/07/09 20:34:57 [notice] 1#1: start worker process 30
2025/07/09 20:34:57 [notice] 1#1: start worker process 31
2025/07/09 20:34:57 [notice] 1#1: start worker process 32
2025/07/09 20:34:57 [notice] 1#1: start worker process 33

$ kubectl get events --field-selector=involvedObject.name=nginx
LAST SEEN   TYPE     REASON      OBJECT      MESSAGE
48s         Normal   Scheduled   pod/nginx   Successfully assigned default/nginx to ubuntu
47s         Normal   Pulled      pod/nginx   Container image "nginx:alpine" already present on machine
47s         Normal   Created     pod/nginx   Created container: nginx
46s         Normal   Started     pod/nginx   Started container nginx

$ kubectl top node
NAME     CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
ubuntu   337m         8%       1618Mi          42%

$ kubectl top pod
NAME    CPU(cores)   MEMORY(bytes)
nginx   0m           4Mi

$ kubectl delete pods nginx
pod "nginx" deleted

```

## PRACTICE #4 - MICROK8S - UBUNTU 24

```bash
$ kubectl api-resources --no-headers | fgrep replicasets
replicasets                         rs         apps/v1                           true    ReplicaSet

$ kubectl explain replicasets
GROUP:      apps
KIND:       ReplicaSet
VERSION:    v1

DESCRIPTION:
    ReplicaSet ensures that a specified number of pod replicas are running at
    any given time
...

$ kubectl create deployment nginx --dry-run=client --output=yaml --image=nginx:alpine --replicas=3 | yq '.kind="ReplicaSet"' | kubectl-neat | tee replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
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

$ kubectl apply --filename=replicaset.yaml
replicaset.apps/nginx created

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   3         3         0       5s

$ kubectl get pods --selector=app=nginx
NAME          READY   STATUS    RESTARTS   AGE
nginx-5hvcg   1/1     Running   0          10s
nginx-dn97b   1/1     Running   0          10s
nginx-ttmlv   1/1     Running   0          10s

$ kubectl describe pods --selector=app=nginx | fgrep Controlled
Controlled By:  ReplicaSet/nginx
Controlled By:  ReplicaSet/nginx
Controlled By:  ReplicaSet/nginx

$ kubectl scale replicaset nginx --replicas=9
replicaset.apps/nginx scaled

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   9         9         9       2m26s

$ kubectl get pods --selector=app=nginx
NAME          READY   STATUS    RESTARTS   AGE
nginx-5hvcg   1/1     Running   0          2m34s
nginx-74w9p   1/1     Running   0          11s
nginx-7bqlw   1/1     Running   0          11s
nginx-dn97b   1/1     Running   0          2m34s
nginx-n747d   1/1     Running   0          11s
nginx-pzczl   1/1     Running   0          11s
nginx-qwnv4   1/1     Running   0          11s
nginx-s27h6   1/1     Running   0          11s
nginx-ttmlv   1/1     Running   0          2m34s

$ kubectl edit replicasets.apps nginx
replicaset.apps/nginx edited

$ kubectl scale replicaset nginx --replicas=0
replicaset.apps/nginx scaled

$ kubectl get replicasets.apps nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   0         0         0       4m9s

$ kubectl get pods --selector=app=nginx
No resources found in default namespace.

$ kubectl delete --filename=replicaset.yaml
replicaset.apps "nginx" deleted

$ rm --verbose replicaset.yaml
removed 'replicaset.yaml'

```

## PRACTICE #5 - MICROK8S - UBUNTU 24

```bash
$ kubectl config get-clusters
NAME
microk8s-cluster

$ kubectl config get-users
NAME
admin

$ kubectl config get-contexts
CURRENT   NAME       CLUSTER            AUTHINFO   NAMESPACE
*         microk8s   microk8s-cluster   admin

$ kubectl config current-context
microk8s

$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:16443
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
current-context: microk8s
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED

```

## PRACTICE #6 - MICROK8S - UBUNTU 24

```bash
$ kubectl api-resources --no-headers | fgrep deployments
deployments                         deploy     apps/v1                           true    Deployment

$ kubectl explain deployments
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
...

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

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           31s

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6b66fbbd46-95hxb   1/1     Running   0          46s
nginx-6b66fbbd46-r7kw8   1/1     Running   0          46s
nginx-6b66fbbd46-zs5pr   1/1     Running   0          46s

$ kubectl get replicasets.apps --selector=app=nginx
NAME               DESIRED   CURRENT   READY   AGE
nginx-6b66fbbd46   3         3         3       2m22s

$ kubectl describe replicasets.apps --selector=app=nginx | fgrep Controlled
Controlled By:  Deployment/nginx

$ kubectl describe deployments.apps nginx | fgrep RollingUpdateStrategy
RollingUpdateStrategy:  25% max unavailable, 25% max surge

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl scale deployment nginx --replicas=9
deployment.apps/nginx scaled

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   9/9     9            9           7m46s

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl set image deployments nginx nginx=nginx:bookworm
deployment.apps/nginx image updated

$ kubectl rollout pause deployment nginx
deployment.apps/nginx paused

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   12/9    5            12          10m

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.containers[].image
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:alpine
nginx:alpine
nginx:alpine
nginx:alpine
nginx:alpine
nginx:alpine
nginx:alpine

$ kubectl rollout resume deployment nginx
deployment.apps/nginx resumed

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   9/9     9            9           11m

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.containers[].image
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm
nginx:bookworm

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine
2         nginx:alpine

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:bookworm
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine
2         nginx:bookworm

$ kubectl rollout undo deployment nginx --to-revision=1
deployment.apps/nginx rolled back

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   7/9     5            7           11m

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
2         nginx:bookworm
3         nginx:alpine

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'

```

## PRACTICE #7 - MICROK8S - UBUNTU 24

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

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           8s

$ kubectl get pods --output=wide --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE    IP             NODE     NOMINATED NODE   READINESS GATES
nginx-6b66fbbd46-5mm5r   1/1     Running   0          113s   10.1.243.223   ubuntu   <none>           <none>
nginx-6b66fbbd46-f69ws   1/1     Running   0          113s   10.1.243.213   ubuntu   <none>           <none>
nginx-6b66fbbd46-k82dn   1/1     Running   0          113s   10.1.243.202   ubuntu   <none>           <none>

$ curl --fail --show-error --silent http://10.1.243.202
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ kubectl expose deployment nginx --port=80
service/nginx exposed

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE    SELECTOR
nginx   ClusterIP   10.152.183.176   <none>        80/TCP    2m5s   app=nginx

$ curl --fail --show-error --silent http://10.152.183.176
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ kubectl delete services nginx
service "nginx" deleted

$ kubectl api-resources --no-headers | fgrep services | fgrep --invert-match api
services                            svc        v1                                true    Service

$ kubectl explain services
KIND:       Service
VERSION:    v1

DESCRIPTION:
    Service is a named abstraction of software service (for example, mysql)
    consisting of local port (for example 3306) that the proxy listens on, and
    the selector that determines which pods will answer requests sent through
    the proxy.
...

$ kubectl create service clusterip nginx --dry-run=client --output=yaml --tcp=80 | kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - name: "80"
    port: 80
  selector:
    app: nginx

$ kubectl apply --filename=service.yaml
service/nginx created

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx   ClusterIP   10.152.183.69   <none>        80/TCP    17s   app=nginx

$ curl --fail --show-error --silent http://10.152.183.69
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ kubectl run alpine --image=alpine:latest --restart=Never --rm --stdin --tty -- nslookup nginx.default.svc.cluster.local
Server:		10.152.183.10
Address:	10.152.183.10:53

Name:	nginx.default.svc.cluster.local
Address: 10.152.183.69


pod "alpine" deleted

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local qemu.lan
nameserver 10.152.183.10
options ndots:5
pod "alpine" deleted

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget -qO- nginx
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
pod "alpine" deleted

$ kubectl delete --filename=service.yaml
deployment.apps "nginx" deleted

$ kubectl create service nodeport nginx --dry-run=client --output=yaml --tcp=80 | kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - name: "80"
    port: 80
  selector:
    app: nginx
  type: NodePort

$ kubectl apply --filename=service.yaml
service/nginx created

$ kubectl get services nginx --output=wide
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
nginx   NodePort   10.152.183.47   <none>        80:31284/TCP   5s    app=nginx

$ curl --fail --ipv4 --show-error --silent http://localhost:31284
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ ip address show dev enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:00:00:e1 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.225/24 metric 100 brd 172.16.0.255 scope global dynamic enp0s3
       valid_lft 2686sec preferred_lft 2686sec
    inet6 fd05:4e60:543f:dcc1:5054:ff:fe00:e1/64 scope global mngtmpaddr noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe00:e1/64 scope link
       valid_lft forever preferred_lft forever

$ curl --fail --show-error --silent http://172.16.0.225:31284
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ kubectl delete --filename=service.yaml
deployment.apps "nginx" deleted

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml service.yaml
removed 'deployment.yaml'
removed 'service.yaml'

```

## PRACTICE #8 - MICROK8S - UBUNTU 24

```bash
$ kubectl create service externalname ipinfo --dry-run=client --external-name=ipinfo.io --output=yaml |  yq 'del(.metadata.labels) | del(.spec.selector)' | kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ipinfo
spec:
  externalName: ipinfo.io
  type: ExternalName

$ kubectl apply --filename=service.yaml
service/ipinfo created

$ kubectl get services ipinfo --output=wide
NAME     TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
ipinfo   ExternalName   <none>       ipinfo.io    <none>    19s   <none>

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget -qO- ipinfo.io
{
  "ip": "82.64.133.21",
  "hostname": "82-64-133-21.subs.proxad.net",
  "city": "Montrichard",
  "region": "Centre",
  "country": "FR",
  "loc": "47.3431,1.1865",
  "org": "AS12322 Free SAS",
  "postal": "41400",
  "timezone": "Europe/Paris",
  "readme": "https://ipinfo.io/missingauth"
}pod "alpine" deleted

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- nslookup ipinfo.default.svc.cluster.local
Server:		10.152.183.10
Address:	10.152.183.10:53

ipinfo.default.svc.cluster.local	canonical name = ipinfo.io

ipinfo.default.svc.cluster.local	canonical name = ipinfo.io
Name:	ipinfo.io
Address: 34.117.59.81

pod "alpine" deleted

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget -qO- ipinfo.default.svc.cluster.local
wget: server returned error: HTTP/1.1 404 Not Found
pod "alpine" deleted
pod default/alpine terminated (Error)

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget --header 'Host: ipinfo.io' -qO- ipinfo.default.svc.cluster.local
{
  "ip": "82.64.133.21",
  "hostname": "82-64-133-21.subs.proxad.net",
  "city": "Montrichard",
  "region": "Centre",
  "country": "FR",
  "loc": "47.3431,1.1865",
  "org": "AS12322 Free SAS",
  "postal": "41400",
  "timezone": "Europe/Paris",
  "readme": "https://ipinfo.io/missingauth"
}pod "alpine" deleted

$ kubectl run alpine --image=alpine:latest --rm --restart=Never --stdin --tty -- wget --header 'Host: ipinfo.io' -qO- ipinfo
{
  "ip": "82.64.133.21",
  "hostname": "82-64-133-21.subs.proxad.net",
  "city": "Montrichard",
  "region": "Centre",
  "country": "FR",
  "loc": "47.3431,1.1865",
  "org": "AS12322 Free SAS",
  "postal": "41400",
  "timezone": "Europe/Paris",
  "readme": "https://ipinfo.io/missingauth"
}pod "alpine" deleted

$ kubectl delete --filename=service.yaml
deployment.apps "ipinfo" deleted

$ rm --verbose service.yaml
removed 'service.yaml'

```

```bash
$ kubectl create deployment nginx --image=nginx:alpine --replicas=3
deployment.apps/nginx created

$ kubectl get pods --output=wide --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE    IP             NODE     NOMINATED NODE   READINESS GATES
nginx-6b66fbbd46-4dbgj   1/1     Running   0          119s   10.1.243.193   ubuntu   <none>           <none>
nginx-6b66fbbd46-4j8bg   1/1     Running   0          119s   10.1.243.237   ubuntu   <none>           <none>
nginx-6b66fbbd46-x4t45   1/1     Running   0          119s   10.1.243.195   ubuntu   <none>           <none>

$ kubectl expose deployment nginx --port=80
service/nginx exposed

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx   ClusterIP   10.152.183.244   <none>        80/TCP    9s    app=nginx

$ curl --fail --show-error --silent http://10.152.183.244
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ kubectl get endpoints nginx
NAME    ENDPOINTS                                         AGE
nginx   10.1.243.193:80,10.1.243.195:80,10.1.243.237:80   4m1s

$ kubectl scale deployment nginx --replicas=1
deployment.apps/nginx scaled

$ kubectl get endpoints nginx
NAME    ENDPOINTS         AGE
nginx   10.1.243.237:80   6m10s

$ kubectl scale deployment nginx --replicas=9
deployment.apps/nginx scaled

$ kubectl get endpoints nginx
NAME    ENDPOINTS         AGE
nginx   10.1.243.237:80   10m

$ kubectl get endpoints nginx
NAME    ENDPOINTS                                                     AGE
nginx   10.1.243.234:80,10.1.243.235:80,10.1.243.237:80 + 3 more...   10m

$ kubectl get endpoints nginx
NAME    ENDPOINTS                                                     AGE
nginx   10.1.243.234:80,10.1.243.235:80,10.1.243.236:80 + 6 more...   11m

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted

$ kubectl delete services nginx
service "nginx" deleted

```

## PRACTICE #9 - MICROK8S - UBUNTU 24

```bash
$ kubectl create deployment nginx --image=nginx:alpine --replicas=3
deployment.apps/nginx created

$ kubectl get pods --selector=app=nginx --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-6b66fbbd46-6mhrp   1/1     Running   0          68s   app=nginx,pod-template-hash=6b66fbbd46
nginx-6b66fbbd46-mn4vr   1/1     Running   0          68s   app=nginx,pod-template-hash=6b66fbbd46
nginx-6b66fbbd46-vmmfs   1/1     Running   0          68s   app=nginx,pod-template-hash=6b66fbbd46

$ kubectl get deployments.apps nginx --show-labels
NAME    READY   UP-TO-DATE   AVAILABLE   AGE    LABELS
nginx   3/3     3            3           111s   app=nginx

$ kubectl get deployments.apps nginx --show-labels
NAME    READY   UP-TO-DATE   AVAILABLE   AGE     APP
nginx   3/3     3            3           3m22s   nginx

$ kubectl label deployments.apps nginx team=development
deployment.apps/nginx labeled

$ kubectl get deployments.apps nginx --label-columns=team
NAME    READY   UP-TO-DATE   AVAILABLE   AGE     TEAM
nginx   3/3     3            3           5m26s   development

$ kubectl label --overwrite deployments.apps nginx team=production
deployment.apps/nginx labeled

$ kubectl get deployments.apps nginx --label-columns=team
NAME    READY   UP-TO-DATE   AVAILABLE   AGE     TEAM
nginx   3/3     3            3           6m37s   production

$ kubectl label deployments.apps nginx team-
deployment.apps/nginx unlabeled

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted

```
