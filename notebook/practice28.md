# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/security/service-accounts

https://www.youtube.com/watch?v=BdXhAvXDqWY&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #28 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep serviceaccounts
serviceaccounts                     sa         v1                                true    ServiceAccount

$ kubectl explain serviceaccounts --output=plaintext-openapiv2
|...|

$ kubectl explain pods.spec.serviceAccountName --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=yaml | kubectl-neat | tee pod.yaml
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

$ kubeconform --verbose pod.yaml
pod.yaml - Pod nginx is valid

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl exec nginx -- ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt
namespace
token

$ kubectl exec nginx -- cat /var/run/secrets/kubernetes.io/serviceaccount/token |
jq --raw-input --raw-output 'split(".")[1] | @base64d | fromjson | .sub'
system:serviceaccount:default:default

$ kubectl create serviceaccount nginx --dry-run=client --output=yaml | kubectl-neat | tee serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx

$ kubeconform --verbose serviceaccount.yaml
serviceaccount.yaml - ServiceAccount nginx is valid

$ kubectl apply --filename=serviceaccount.yaml
serviceaccount/nginx created

$ kubectl get serviceaccounts nginx
NAME    AGE
nginx   15s

$ kubectl delete --filename=pod.yaml
pod "nginx" deleted from default namespace

$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=yaml |
yq '.spec.serviceAccountName = "nginx"' | kubectl-neat | tee pod.yaml
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
  serverAccountNmae: nginx

$ kubeconform --verbose pod.yaml
pod.yaml - Pod nginx is valid

$ kubectl apply --filename=pod.yaml
pod/nginx created

$ kubectl exec nginx -- cat /var/run/secrets/kubernetes.io/serviceaccount/token |
jq --raw-input --raw-output 'split(".")[1] | @base64d | fromjson | .sub'
system:serviceaccount:default:nginx

$ kubectl delete --filename=pod.yaml
pod "nginx" deleted from default namespace

$ kubectl delete --filename=serviceaccount.yaml
serviceaccount "nginx" deleted from default namespace

$ rm --verbose pod.yaml serviceaccount.yaml
removed 'pod.yaml'
removed 'serviceaccount.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
