# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_config

https://www.youtube.com/watch?v=xX14RxILY0Q&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap


## PRACTICE #4 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

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

$ kubectl config get-contexts --no-headers
*     microk8s   microk8s-cluster   admin

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

$ kubectl config view --output=jsonpath='{.current-context}' && echo
microk8s
```

## PRACTICE #4 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl config get-clusters
NAME
kind-cluster

$ kubectl config get-users
NAME
kind-cluster

$ kubectl config get-contexts
CURRENT   NAME           CLUSTER        AUTHINFO       NAMESPACE
*         kind-cluster   kind-cluster   kind-cluster

$ kubectl config get-contexts --no-headers
*     kind-cluster   kind-cluster   kind-cluster

$ kubectl config current-context
kind-cluster

$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:40523
  name: kind-cluster
contexts:
- context:
    cluster: kind-cluster
    user: kind-cluster
  name: kind-cluster
current-context: kind-cluster
kind: Config
preferences: {}
users:
- name: kind-cluster
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED

$ kubectl config view --output=jsonpath='{.current-context}' && echo
kind-cluster
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
