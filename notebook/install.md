# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm  
https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview  
https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries

https://www.youtube.com/watch?v=23MhuIVYMHE&list=PLn6POgpklwWo4TTIjWOnYGmw2MHDN1HAL  
https://www.youtube.com/watch?v=buRvk-Atyes&list=PLn6POgpklwWo4TTIjWOnYGmw2MHDN1HAL

## INSTALL - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
```

## INSTALL - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
```

## INSTALL - KUBERNETES - KUBEADM - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kubeadm](img/kubeadm.webp "Kubeadm")](https://kubernetes.io/docs/reference/setup-tools/kubeadm)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ curl --fail --location --show-error --silent https://dl.k8s.io/release/stable.txt && echo
v1.34.1

$ curl --fail --location --show-error --silent https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key |
sudo dd of=/etc/apt/trusted.gpg.d/kubernetes.asc status=none

$ sudo dd of=/etc/apt/sources.list.d/kubernetes.list status=none <<EOF
deb [arch=amd64] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /
EOF

$ sudo apt --quiet=2 update
All packages are up to date.
```

```bash
$ apt --option=Apt::Cmd::Disable-Script-Warning=true policy kubectl | head --lines=6
kubectl:
  Installed: (none)
  Candidate: 1.34.1-1.1
  Version table:
     1.34.1-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages

$ apt depends kubectl
kubectl

$ sudo apt install --no-install-recommends --quiet=2 --yes kubectl
The following NEW packages will be installed:
  kubectl
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B/11.7 MB of archives.
|...|

$ ls --width=1 /usr/share/doc/kubectl
LICENSE
README.md

$ whereis kubectl
kubectl: /usr/bin/kubectl

$ kubectl version --client
Client Version: v1.34.1
Kustomize Version: v5.7.1
```

```bash
$ apt --option=Apt::Cmd::Disable-Script-Warning=true policy kubeadm | head --lines=6
kubeadm:
  Installed: 1.34.1-1.1
  Candidate: 1.34.1-1.1
  Version table:
 *** 1.34.1-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages

$ sudo apt install --no-install-recommends --quiet=2 --yes kubeadm
The following additional packages will be installed:
  cri-tools
The following NEW packages will be installed:
  cri-tools kubeadm
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B/29.2 MB of archives.

$ apt depends kubeadm
kubeadm
  Depends: cri-tools (>= 1.30.0)

$ ls --width=1 /usr/share/doc/kubeadm
LICENSE
README.md

$ whereis kubeadm
kubeadm: /usr/bin/kubeadm

$ kubeadm version --output=short
v1.34.1

$ kubeadm version --output=yaml | yq
clientVersion:
  buildDate: "2025-09-09T19:43:15Z"
  compiler: gc
  gitCommit: 93248f9ae092f571eb870b7664c534bfc7d00f03
  gitTreeState: clean
  gitVersion: v1.34.1
  goVersion: go1.24.6
  major: "1"
  minor: "34"
  platform: linux/amd64
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
