# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/access-authn-authz/authorization  
https://kubernetes.io/docs/reference/access-authn-authz/rbac

https://www.youtube.com/watch?v=gkOMeqUV52U&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #29- KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep roles | fgrep true
roles                                          rbac.authorization.k8s.io/v1      true    Role

$ kubectl api-resources --no-headers | fgrep rolebindings | fgrep true
rolebindings                                   rbac.authorization.k8s.io/v1      true    RoleBinding

$ kubectl explain roles --output=plaintext-openapiv2
|...|

$ kubectl explain roles.rules --output=plaintext-openapiv2
|...|

$ kubectl explain rolebindings --output=plaintext-openapiv2
|...|

$ kubectl explain rolebindings.roleRef --output=plaintext-openapiv2
|...|

$ kubectl explain rolebindings.subjects --output=plaintext-openapiv2
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

$ kubectl auth can-i --as=system:serviceaccount:default:nginx get pods
no

$ kubectl auth can-i --as=system:serviceaccount:default:nginx list pods
no

$ kubectl auth can-i --as=system:serviceaccount:default:nginx watch pods
no

$ kubectl api-resources --no-headers --output=yaml | yq '.resources.[] | select(.kind == "Pod")'
categories:
  - all
kind: Pod
name: pods
namespaced: true
shortNames:
  - po
singularName: pod
verbs:
  - create
  - delete
  - deletecollection
  - get
  - list
  - patch
  - update
  - watch
version: v1

$ kubectl create role nginx --verb=get,list --resource=pods --dry-run=client --output=yaml |
kubectl-neat | tee role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: nginx
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list

$ kubeconform --verbose role.yaml
role.yaml - Role nginx is valid

$ kubectl apply --filename=role.yaml
role.rbac.authorization.k8s.io/nginx created

$ kubectl create rolebinding nginx --role=nginx --serviceaccount=default:nginx --output=yaml \
--dry-run=client | kubectl-neat | tee rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx
subjects:
- kind: ServiceAccount
  name: nginx
  namespace: default

$ kubeconform --verbose rolebinding.yaml
rolebinding.yaml - RoleBinding nginx is valid

$ kubectl apply --filename=rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/nginx created

$ kubectl auth can-i --as=system:serviceaccount:default:nginx --list --no-headers | fgrep pods |
tr --squeeze-repeats ' '
pods [] [] [get list]

$ kubectl auth can-i --as=system:serviceaccount:default:nginx get pods
yes

$ kubectl auth can-i --as=system:serviceaccount:default:nginx list pods
yes

$ kubectl auth can-i --as=system:serviceaccount:default:nginx watch pods
no

$ kubectl get pods --as=system:serviceaccount:default:nginx
No resources found in default namespace.

$ kubectl delete --filename=rolebinding.yaml
rolebinding.rbac.authorization.k8s.io "nginx" deleted from default namespace

$ kubectl delete --filename=role.yaml
role.rbac.authorization.k8s.io "nginx" deleted from default namespace

$ kubectl delete --filename=serviceaccount.yaml
serviceaccount "nginx" deleted from default namespace

$ rm --verbose serviceaccount.yaml role.yaml rolebinding.yaml
removed 'serviceaccount.yaml'
removed 'role.yaml'
removed 'rolebinding.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
