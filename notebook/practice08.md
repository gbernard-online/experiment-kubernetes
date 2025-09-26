# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_label

https://www.youtube.com/watch?v=cMzEl3I9qsc&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #8 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl create deployment nginx --image=nginx:alpine --replicas=3
deployment.apps/nginx created

$ kubectl get pods --selector=app=nginx --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-7977cdf8f5-6csrf   1/1     Running   0          5s    app=nginx,pod-template-hash=7977cdf8f5
nginx-7977cdf8f5-9rf6t   1/1     Running   0          5s    app=nginx,pod-template-hash=7977cdf8f5
nginx-7977cdf8f5-xwdr2   1/1     Running   0          5s    app=nginx,pod-template-hash=7977cdf8f5

$ kubectl get deployments.apps nginx --show-labels
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
nginx   3/3     3            3           21s   app=nginx

$ kubectl label deployments.apps nginx priority=medium
deployment.apps/nginx labeled

$ kubectl get deployments.apps nginx --label-columns=priority
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   PRIORITY
nginx   3/3     3            3           65s   medium

$ kubectl label --overwrite deployments.apps nginx priority=high
deployment.apps/nginx labeled

$ kubectl get deployments.apps nginx --label-columns=priority
NAME    READY   UP-TO-DATE   AVAILABLE   AGE    PRIORITY
nginx   3/3     3            3           117s   high

$ kubectl label deployments.apps nginx priority-
deployment.apps/nginx unlabeled

$ kubectl get deployments.apps nginx --label-columns=priority
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   PRIORITY
nginx   3/3     3            3           60s

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
