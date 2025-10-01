# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset

https://www.youtube.com/watch?v=BdXhAvXDqWY&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #27 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep daemonsets
daemonsets                          ds         apps/v1                           true    DaemonSet

$ kubectl explain daemonsets --output=plaintext-openapiv2
|...|

$ kubectl explain daemonsets.spec --output=plaintext-openapiv2
|...|

$ kubectl explain daemonsets.spec.template.spec.hostNetwork --output=plaintext-openapiv2
|...|
```

```bash
$ echo 'apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
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
      hostNetwork: true' | kubectl-neat | tee daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
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
      hostNetwork: true

$ kubeconform -verbose daemonset.yaml
daemonset.yaml - DaemonSet nginx is valid

$ kubectl apply --filename=daemonset.yaml
daemonset.apps/nginx created

$ kubectl annotate daemonsets.apps nginx kubernetes.io/change-cause=nginx:alpine
daemonset.apps/nginx annotated

$ kubectl rollout history daemonset nginx
daemonset.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>

$ kubectl get daemonsets.apps nginx
NAME    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
nginx   3         3         3       3            3           <none>          31s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-yellow
cluster-worker-green
cluster-worker-red

$ docker inspect --format='{{.NetworkSettings.Networks.kind.IPAddress}}' \
cluster-worker-green cluster-worker-red cluster-worker-yellow
172.17.1.3
172.17.1.4
172.17.1.5

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.3
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ curl --connect-timeout 5 --fail --show-error --silent 172.17.1.5
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl delete --filename=daemonset.yaml
daemonset.apps "nginx" deleted from default namespace

$ rm --verbose daemonset.yaml
removed 'daemonset.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
