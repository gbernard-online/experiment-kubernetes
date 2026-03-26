# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes

https://www.youtube.com/watch?v=9NVF-vUVYrw&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=qTspluN-uNI&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=SNWaRyKjqbE&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #31 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.containers.startupProbe --output=plaintext-openapiv2
|...|

$ kubectl explain pods.spec.containers.livenessProbe --output=plaintext-openapiv2
|...|

$ kubectl explain pods.spec.containers.readinessProbe --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=json --restart=Never |
jq '.spec.containers[0].startupProbe = input' - <(echo '{
  "httpGet": {
    "port":8080
  },
  "initialDelaySeconds": 30,
  "periodSeconds": 15
}') | kubectl-neat --output=yaml | tee pod.yaml
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
    startupProbe:
      httpGet:
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 15
  restartPolicy: Never

$ kubeconform -verbose pod.yaml
pod.yaml - Pod nginx is valid

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Running   0          6s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
false
false

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Running   0          37s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
false
false

$ kubectl get pods nginx
NAME    READY   STATUS      RESTARTS   AGE
nginx   0/1     Completed   0          65s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
false
false

$ kubectl delete --filename=pod.yaml
pod "nginx" deleted from default namespace

$ patch --forward --reject-file=- pod.yaml <<EOF
13c13
<         port: 8080
---
>         port: 80
EOF
patching file pod.yaml

$ kubeconform -verbose pod.yaml
pod.yaml - Pod nginx is valid

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Running   0          7s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
false
false

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          34s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
true
true

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          78s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
true
true

$ kubectl delete --filename=pod.yaml
pod "nginx" deleted from default namespace

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=json --restart=Never |
jq '.spec.containers[0].livenessProbe = input' - <(echo '{
  "httpGet": {
    "port":8080
  },
  "initialDelaySeconds": 30,
  "periodSeconds": 15
}') | kubectl-neat --output=yaml | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    livenessProbe:
      httpGet:
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 15
    name: nginx
  restartPolicy: Never

$ kubeconform -verbose pod.yaml
pod.yaml - Pod nginx is valid

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          6s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
true
true

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          37s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
true
true

$ kubectl get pods nginx
NAME    READY   STATUS      RESTARTS   AGE
nginx   0/1     Completed   0          68s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
false
false

$ kubectl delete --filename=pod.yaml
pod "nginx" deleted from default namespace

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=json --restart=Never |
jq '.spec.containers[0].readinessProbe = input' - <(echo '{
  "httpGet": {
    "port":8080
  },
  "initialDelaySeconds": 30,
  "periodSeconds": 15
}') | kubectl-neat --output=yaml | tee pod.yaml
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
    readinessProbe:
      httpGet:
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 15
  restartPolicy: Never

$ kubeconform -verbose pod.yaml
pod.yaml - Pod nginx is valid

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Running   0          8s

$ kubectl get pods nginx --output=yaml | ya '.status.containerStatuses[0] | (.started,.ready)'
true
false

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Running   0          37s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
true
false

$ kubectl get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Running   0          77s

$ kubectl get pods nginx --output=yaml | yq '.status.containerStatuses[0] | (.started,.ready)'
true
false

$ kubectl delete --filename=pod.yaml
pod "nginx" deleted from default namespace

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
