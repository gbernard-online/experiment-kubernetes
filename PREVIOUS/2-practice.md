## PRACTICE #9 - MICROK8S - UBUNTU 24

https://www.youtube.com/watch?v=5b3kkJ0pUjA&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap&index=27

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
