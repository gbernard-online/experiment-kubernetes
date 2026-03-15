# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=gkOMeqUV52U&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #29- KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep clusterroles
clusterroles                                   rbac.authorization.k8s.io/v1      false   ClusterRole

$ kubectl api-resources --no-headers | fgrep clusterrolebindings
clusterrolebindings                            rbac.authorization.k8s.io/v1      false   ClusterRoleBinding

$ kubectl explain clusterroles
|...|

$ kubectl explain clusterrolebindings
|...|
```

```bash
$ kubectl create serviceaccount nginx --dry-run=client --output=yaml |
kubectl-neat | tee serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx

$ kubeconform --verbose serviceaccount.yaml
serviceaccount.yaml - ServiceAccount nginx is valid

$ kubectl apply --filename=serviceaccount.yaml
serviceaccount/nginx created

$ kubectl auth can-i --all-namespaces --as=system:serviceaccount:default:nginx get namespaces
no

$ kubectl auth can-i --all-namespaces --as=system:serviceaccount:default:nginx list namespaces
no

$ kubectl auth can-i --all-namespaces --as=system:serviceaccount:default:nginx watch namespaces
no

$ kubectl api-resources --no-headers --output=yaml | yq '.resources.[] | select(.kind == "Namespace")'
kind: Namespace
name: namespaces
namespaced: false
shortNames:
  - ns
singularName: namespace
verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
version: v1

$ kubectl create clusterrole nginx --verb=get,list --resource=namespaces --dry-run=client --output=yaml |
kubectl-neat | tee clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nginx
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
  - list

$ kubeconform --verbose clusterrole.yaml
clusterrole.yaml - ClusterRole nginx is valid

$ kubectl apply --filename=clusterrole.yaml
clusterrole.rbac.authorization.k8s.io/nginx created

$ kubectl create clusterrolebinding nginx --clusterrole=nginx --serviceaccount=default:nginx --output=yaml \
--dry-run=client | kubectl-neat | tee clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx
subjects:
- kind: ServiceAccount
  name: nginx
  namespace: default

$ kubeconform --verbose clusterrolebinding.yaml
clusterrolebinding.yaml - ClusterRoleBinding nginx is valid

$ kubectl apply --filename=clusterrolebinding.yaml
clusterrolebinding.rbac.authorization.k8s.io/nginx created

$ kubectl auth can-i --as=system:serviceaccount:default:nginx --list | fgrep namespaces |
tr --squeeze-repeats ' '
namespaces [] [] [get list]

$ kubectl auth can-i --all-namespaces --as=system:serviceaccount:default:nginx get namespaces
yes

$ kubectl auth can-i --all-namespaces --as=system:serviceaccount:default:nginx list namespaces
yes

$ kubectl auth can-i --all-namespaces --as=system:serviceaccount:default:nginx watch namespaces
no

$ kubectl get namespaces --as=system:serviceaccount:default:nginx default --output=yaml | kubectl-neat
apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: default
  name: default
spec:
  finalizers:
  - kubernetes

$ kubectl delete --filename=clusterrolebinding.yaml
clusterrolebinding.rbac.authorization.k8s.io "nginx" deleted

$ kubectl delete --filename=clusterrole.yaml
clusterrole.rbac.authorization.k8s.io "nginx" delete

$ kubectl delete --filename=serviceaccount.yaml
serviceaccount "nginx" deleted from default namespace

$ rm --verbose serviceaccount.yaml clusterrole.yaml clusterrolebinding.yaml
removed 'serviceaccount.yaml'
removed 'clusterrole.yaml'
removed 'clusterrolebinding.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
