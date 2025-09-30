# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/controllers/deployment  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_annotate  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_rollout

https://www.youtube.com/watch?v=AFEU_mBbzr0&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=WSfB7Ae4SPg&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #5 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep deployments
deployments                         deploy     apps/v1                           true    Deployment

$ kubectl explain deployments --output=plaintext-openapiv2
|...|

$ kubectl explain deployments.spec --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=yaml --replicas=3 |
kubectl-neat | tee deployment.yaml
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
nginx-7977cdf8f5-9n7p6   1/1     Running   0          39s
nginx-7977cdf8f5-bjnt6   1/1     Running   0          39s
nginx-7977cdf8f5-mczcq   1/1     Running   0          39s

$ kubectl get replicasets.apps --selector=app=nginx
NAME               DESIRED   CURRENT   READY   AGE
nginx-7977cdf8f5   3         3         3       70s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-yellow
cluster-worker-green

$ kubectl describe replicasets.apps --selector=app=nginx | fgrep Controlled
Controlled By:  Deployment/nginx

$ kubectl get deployments.apps nginx --output=yaml | yq .spec.strategy
rollingUpdate:
  maxSurge: 25%
  maxUnavailable: 25%
type: RollingUpdate

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>

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
nginx   9/9     9            9           2m8s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-yellow
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-green

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
nginx   7/9     5            7           3m11s

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
nginx   8/9     5            8           6m1s

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

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
