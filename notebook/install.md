# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm  
https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview  
https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries

https://www.youtube.com/watch?v=23MhuIVYMHE&list=PLn6POgpklwWo4TTIjWOnYGmw2MHDN1HAL  
https://www.youtube.com/watch?v=buRvk-Atyes&list=PLn6POgpklwWo4TTIjWOnYGmw2MHDN1HAL

## INSTALL - KUBERNETES - KUBEADM - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kubeadm](img/kubeadm.webp "Kubeadm")](https://kubernetes.io/docs/reference/setup-tools/kubeadm)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ curl --fail --location --show-error --silent https://dl.k8s.io/release/stable.txt && echo
v1.34.1

$ curl --fail --location --show-error --silent https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key |
sudo dd of=/etc/apt/trusted.gpg.d/kubernetes.asc status=none

$ sudo dd of=/etc/apt/sources.list.d/kubernetes.list status=none <<EOF
deb [arch=amd64] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /
EOF

$ sudo apt --quiet=2 update
All packages are up to date.
```

```bash
$ apt-cache madison kubectl
   kubectl | 1.33.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubectl | 1.33.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubectl | 1.33.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubectl | 1.33.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubectl | 1.33.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubectl | 1.33.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages

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
Client Version: v1.33.5
Kustomize Version: v5.6.0

$ kubectl version --client --output=yaml | yq
clientVersion:
  buildDate: "2025-09-09T19:52:31Z"
  compiler: gc
  gitCommit: 03e764d0394bdff662e960c70d25b3c30d731666
  gitTreeState: clean
  gitVersion: v1.33.5
  goVersion: go1.24.6
  major: "1"
  minor: "33"
  platform: linux/amd64
kustomizeVersion: v5.6.0
```

```bash
$ apt-cache madison kubeadm
   kubeadm | 1.33.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages

$ apt depends kubeadm
kubeadm
  Depends: cri-tools (>= 1.30.0)

The following additional packages will be installed:
  cri-tools
The following NEW packages will be installed:
  cri-tools kubeadm
0 upgraded, 2 newly installed, 0 to remove and 1 not upgraded.
Need to get 30.0 MB of archives.
|...|

$ ls --width=1 /usr/share/doc/kubeadm
LICENSE
README.md

$ whereis kubeadm
kubeadm: /usr/bin/kubeadm

$ kubeadm version --output=short
v1.33.5

$ kubeadm version --output=yaml | yq
clientVersion:
  buildDate: "2025-09-09T19:50:45Z"
  compiler: gc
  gitCommit: 03e764d0394bdff662e960c70d25b3c30d731666
  gitTreeState: clean
  gitVersion: v1.33.5
  goVersion: go1.24.6
  major: "1"
  minor: "33"
  platform: linux/amd64
```

## INSTALL - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ sudo snap install --classic microk8s
|...|
microk8s (1.32/stable) v1.32.8 from Canonical✓ installed

$ sudo microk8s version
MicroK8s v1.32.8 revision 8355

$ sudo usermod --append --groups=microk8s user

$ logout
```

```bash
$ microk8s status --wait-ready
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
addons:
  enabled:
    dns                  # (core) CoreDNS
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
  disabled:
    cert-manager         # (core) Cloud native certificate management
    cis-hardening        # (core) Apply CIS K8s hardening
    community            # (core) The community addons repository
    dashboard            # (core) The Kubernetes dashboard
    gpu                  # (core) Alias to nvidia add-on
    host-access          # (core) Allow Pods connecting to Host services smoothly
    hostpath-storage     # (core) Storage class; allocates storage from host directory
    ingress              # (core) Ingress controller for external access
    kube-ovn             # (core) An advanced network fabric for Kubernetes
    mayastor             # (core) OpenEBS MayaStor
    metallb              # (core) Loadbalancer for your Kubernetes cluster
    metrics-server       # (core) K8s Metrics Server for API access to service metrics
    minio                # (core) MinIO object storage
    nvidia               # (core) NVIDIA hardware (GPU and network) support
    observability        # (core) A lightweight observability stack for logs, traces and metrics
    prometheus           # (core) Prometheus operator for monitoring and logging
    rbac                 # (core) Role-Based Access Control for authorisation
    registry             # (core) Private image registry exposed on localhost:32000
    rook-ceph            # (core) Distributed Ceph storage using Rook
    storage              # (core) Alias to hostpath-storage add-on, deprecated

$ microk8s kubectl version
Client Version: v1.32.8
Kustomize Version: v5.5.0
Server Version: v1.32.8

$ microk8s kubectl version --output=yaml | yq
clientVersion:
  buildDate: "2025-08-15T16:02:26Z"
  compiler: gc
  gitCommit: 2e83bc4bf31e88b7de81d5341939d5ce2460f46f
  gitTreeState: clean
  gitVersion: v1.32.8
  goVersion: go1.23.11
  major: "1"
  minor: "32"
  platform: linux/amd64
kustomizeVersion: v5.5.0
serverVersion:
  buildDate: "2025-08-15T16:03:33Z"
  compiler: gc
  gitCommit: 2e83bc4bf31e88b7de81d5341939d5ce2460f46f
  gitTreeState: clean
  gitVersion: v1.32.8
  goVersion: go1.23.11
  major: "1"
  minor: "32"
  platform: linux/amd64

$ microk8s kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
ubuntu   Ready    <none>   7m10s   v1.32.8

$ microk8s kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   7m31s

$ microk8s kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
ubuntu   Ready    <none>   8m28s   v1.32.8

$ microk8s kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   9m4s

$ microk8s enable metrics-server
Infer repository core for addon metrics-server
Enabling Metrics-Server
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
clusterrolebinding.rbac.authorization.k8s.io/microk8s-admin created
Metrics-Server is enabled

$ sleep 30
```

```bash
$ microk8s kubectl config view --raw >$HOME/.kube/config

$ kubectl version
Client Version: v1.33.5
Kustomize Version: v5.6.0
Server Version: v1.32.8
```

## INSTALL - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
