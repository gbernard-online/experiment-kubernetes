# EXPERIMENT KUBERNETES

## PURGE - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ sudo snap remove --purge microk8s
microk8s removed

$ rm --verbose $HOME/.kube/config
removed '/home/user/.kube/config'

$ rm --force --recursive $HOME/.kube/cache
```

## PURGE - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kind delete clusters --all
Deleted nodes: ["cluster-control-plane" "cluster-worker-green" "cluster-worker-red" "cluster-worker-yellow"]
Deleted clusters: ["cluster"]

$ rm --verbose $HOME/.kube/config
removed '/home/user/.kube/config'

$ rm --force --recursive $HOME/.kube/cache
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
