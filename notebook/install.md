# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_version  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_get  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_cluster-info  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_wait

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm  
https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-version

https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s  
https://microk8s.io/docs/working-with-kubectl  
https://microk8s.io/docs/command-reference#heading--microk8s-kubectl  
https://microk8s.io/docs/addons

https://kind.sigs.k8s.io/docs/user/configuration  
https://kind.sigs.k8s.io/docs/user/ingress  
https://iamunnip.hashnode.dev/kind-creating-a-kubernetes-cluster-part-1  
https://iamunnip.hashnode.dev/kind-configuring-extra-port-mappings-ipv6-networking-part-4  
https://iamunnip.hashnode.dev/kind-setting-up-an-ingress-controller-part-5

https://www.youtube.com/watch?v=23MhuIVYMHE&list=PLn6POgpklwWo4TTIjWOnYGmw2MHDN1HAL  
https://www.youtube.com/watch?v=buRvk-Atyes&list=PLn6POgpklwWo4TTIjWOnYGmw2MHDN1HAL  
https://www.youtube.com/watch?v=AskMZMtjk2g&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

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
$ apt-cache madison kubectl
   kubectl | 1.34.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
   kubectl | 1.34.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages

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

$ kubectl version --client --output=yaml | yq
clientVersion:
  buildDate: "2025-09-09T19:44:50Z"
  compiler: gc
  gitCommit: 93248f9ae092f571eb870b7664c534bfc7d00f03
  gitTreeState: clean
  gitVersion: v1.34.1
  goVersion: go1.24.6
  major: "1"
  minor: "34"
  platform: linux/amd64
kustomizeVersion: v5.7.1
```

```bash
$ apt-cache madison kubeadm
   kubeadm | 1.34.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
   kubeadm | 1.34.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages

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

```bash
$ sudo mkdir --verbose /opt/kubectl-neat
mkdir: created directory '/opt/kubectl-neat'

$ curl --fail --location --show-error --silent \
https://github.com/itaysk/kubectl-neat/releases/latest/download/kubectl-neat_linux_amd64.tar.gz |
sudo tar --directory=/opt/kubectl-neat --extract --gunzip --verbose
LICENSE
kubectl-neat

$ sudo chown --verbose root: /opt/kubectl-neat/{kubectl-neat,LICENSE}
changed ownership of '/opt/kubectl-neat/kubectl-neat' from 501:staff to root:root
changed ownership of '/opt/kubectl-neat/LICENSE' from 501:staff to root:root

$ sudo ln --symbolic --verbose /opt/kubectl-neat/kubectl-neat /usr/local/bin/kubectl-neat
'/usr/local/bin/kubectl-neat' -> '/opt/kubectl-neat/kubectl-neat'

$ kubectl-neat version
kubectl-neat version: 2.0.4
```

```bash
$ sudo mkdir --verbose /opt/kubeconform
mkdir: created directory '/opt/kubeconform'

$ curl --fail --location --show-error --silent \
https://github.com/yannh/kubeconform/releases/latest/download/kubeconform-linux-amd64.tar.gz |
sudo tar --directory=/opt/kubeconform --extract --gunzip --verbose
LICENSE
kubeconform

$ sudo chown --verbose root: /opt/kubectl-neat/LICENSE
changed ownership of '/opt/kubectl-neat/LICENSE' from 501:staff to root:root

$ sudo ln --symbolic --verbose /opt/kubeconform/kubeconform /usr/local/bin/kubeconform
'/usr/local/bin/kubeconform' -> '/opt/kubeconform/kubeconform'

$ kubeconform -v
v0.7.0
```

```bash
$ apt policy nfs-kernel-server
nfs-kernel-server:
  Installed: (none)
  Candidate: 1:2.6.4-3ubuntu5.1
  Version table:
     1:2.6.4-3ubuntu5.1 500
        500 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1:2.6.4-3ubuntu5 500
        500 http://archive.ubuntu.com/ubuntu noble/main amd64 Packages

$ apt depends nfs-kernel-server
nfs-kernel-server
  Depends: libblkid1 (≥ 2.16)
  Depends: libc6 (≥ 2.38)
  Depends: libcap2 (≥ 1:2.10)
  Depends: libevent-core-2.1-7t64 (≥ 2.1.8-stable)
  Depends: libsqlite3-0 (≥ 3.7.15)
  Depends: libtirpc3t64 (≥ 1.0.2)
  Depends: libuuid1 (≥ 2.31.1)
  Depends: libwrap0 (≥ 7.6-4~)
  Depends: libxml2 (≥ 2.7.4)
  Depends: nfs-common (= 1:2.6.4-3ubuntu5.1)
  Depends: ucf
  Depends: netbase
  Depends: keyutils
  Conflicts: <knfs>
  Conflicts: <nfs-server>
  Recommends: python3
  Recommends: python3-yaml
  Suggests: procps
  Replaces: <knfs>
    nfs-kernel-server
  Replaces: <nfs-server>
    nfs-kernel-server

$ sudo apt install --yes nfs-kernel-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libnfsidmap1 nfs-common rpcbind
Suggested packages:
  watchdog
The following NEW packages will be installed:
  keyutils libnfsidmap1 nfs-common nfs-kernel-server rpcbind
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 569 kB of archives.
|...|

$ sudo mkdir --verbose /srv/nfs
mkdir: created directory '/srv/nfs'

$ sudo chmod --verbose g-rx,o-rx /srv/nfs
mode of '/srv/nfs' changed from 0755 (rwxr-xr-x) to 0700 (rwx------)

$ echo '/srv/nfs *(fsid=root,insecure,no_root_squash,no_subtree_check,rw,sync)' |
sudo tee --append /etc/exports
/srv/nfs *(fsid=root,insecure,no_root_squash,no_subtree_check,rw,sync)

$ sudo systemctl restart nfs-server.service rpcbind.service

$ sudo exportfs -av
exporting *:/srv/nfs

$ systemctl is-active nfs-server.service
active

$ systemctl is-active rpcbind.service
active
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

$ echo '
if [ -x "$(command -v microk8s)" ]; then
  alias kubectl=microk8s.kubectl
  if [ -r /var/snap/microk8s/current/credentials/client.config ]; then
    export KUBECONFIG=/var/snap/microk8s/current/credentials/client.config
  fi
fi' | tee --append $HOME/.bashrc

if [ -x "$(command -v microk8s)" ]; then
  alias kubectl=microk8s.kubectl
  if [ -r /var/snap/microk8s/current/credentials/client.config ]; then
    export KUBECONFIG=/var/snap/microk8s/current/credentials/client.config
  fi
fi

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

$ kubectl version
Client Version: v1.32.8
Kustomize Version: v5.5.0
Server Version: v1.32.8

$ kubectl version --output=yaml | yq
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

$ kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
ubuntu   Ready    <none>   7m10s   v1.32.8

$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   7m31s
```

```bash
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

$ kubectl wait pods --for=condition=ready \
--namespace=kube-system --selector=k8s-app=metrics-server
pod/metrics-server-7dbd8b5cc9-brs2z condition met

$ microk8s status --addon=metrics-server
enabled
```

```bash
$ sudo microk8s enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled

$ kubectl wait pods --for=condition=ready \
--namespace=ingress --selector=name=nginx-ingress-microk8s --timeout=60s
pod/nginx-ingress-microk8s-controller-8wfnl condition met

$ microk8s status --addon=ingress
enabled
```

```bash
$ microk8s enable hostpath-storage
Infer repository core for addon hostpath-storage
Enabling default storage class.
WARNING: Hostpath storage is not suitable for production environments.
         A hostpath volume can grow beyond the size limit set in the volume claim manifest.

deployment.apps/hostpath-provisioner created
storageclass.storage.k8s.io/microk8s-hostpath created
serviceaccount/microk8s-hostpath created
clusterrole.rbac.authorization.k8s.io/microk8s-hostpath created
clusterrolebinding.rbac.authorization.k8s.io/microk8s-hostpath created
Storage will be available soon.

$ kubectl wait pods --for=condition=ready \
--namespace=ingress --selector=name=nginx-ingress-microk8s --timeout=60s
pod/nginx-ingress-microk8s-controller-8wfnl condition met

$ microk8s status --addon=hostpath-storage
enabled
```

## INSTALL - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ sudo mkdir --verbose /opt/kind
mkdir: created directory '/opt/kind'

$ sudo curl --fail --location --output /opt/kind/kind-linux-amd64 --show-error --silent \
https://github.com/kubernetes-sigs/kind/releases/latest/download/kind-linux-amd64

$ sudo chmod --verbose +x /opt/kind/kind-linux-amd64
mode of '/opt/kind/kind-linux-amd64' changed from 0644 (rw-r--r--) to 0755 (rwxr-xr-x)

$ sudo ln --symbolic --verbose /opt/kind/kind-linux-amd64 /usr/local/bin/kind
'/usr/local/bin/kind' -> '/opt/kind/kind-linux-amd64'

$ kind --version
kind version 0.30.0

$ kind completion bash | sudo dd of=/etc/bash_completion.d/kind status=none

$ logout
```

```bash
$ kind create cluster --config=- <<EOF
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
name: cluster
nodes:
- role: control-plane
- kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      name: cluster-worker-green
  labels:
    color: green
  role: worker
- kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      name: cluster-worker-red
  labels:
    color: red
  role: worker
- kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      name: cluster-worker-yellow
  labels:
    color: yellow
  role: worker
EOF
Creating cluster "cluster" ...
 ✓ Ensuring node image (kindest/node:v1.34.0) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-cluster

Thanks for using kind! 😊

$ kubectl cluster-info --context=kind-cluster
Kubernetes control plane is running at https://127.0.0.1:41155
CoreDNS is running at https://127.0.0.1:41155/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl cluster-info dump --context=kind-cluster | jq
|...|

$ kubectl version
Client Version: v1.33.5
Kustomize Version: v5.6.0
Server Version: v1.34.0

$ kubectl get nodes
NAME                    STATUS   ROLES           AGE     VERSION
cluster-control-plane   Ready    control-plane   2m22s   v1.34.0
cluster-worker-green    Ready    <none>          2m9s    v1.34.0
cluster-worker-red      Ready    <none>          2m9s    v1.34.0
cluster-worker-yellow   Ready    <none>          2m9s    v1.34.0

$ docker container ls --format='{{ .Names }} {{ .Ports }}'
cluster-control-plane 127.0.0.1:42207->6443/tcp
cluster-worker
cluster-worker3
cluster-worker2

$ docker container rename cluster-worker cluster-worker-green

$ docker container rename cluster-worker2 cluster-worker-red

$ docker container rename cluster-worker3 cluster-worker-yellow

$ docker container ls --format='{{ .Names }}'
cluster-control-plane
cluster-worker-green
cluster-worker-red
cluster-worker-yellow

$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m34s

$ kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
ubuntu   Ready    <none>   8m28s   v1.32.8

$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   9m4s
```

```bash
$ cat >kustomization.yaml <<EOF
patches:
- patch: |-
    - op: add
      path: /spec/template/spec/containers/0/args/-
      value: --kubelet-insecure-tls
    - op: add
      path: "/spec/template/spec/tolerations"
      value:
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
          operator: Equal
        - effect: NoSchedule
          key: node-role.kubernetes.io/control-plane
          operator: Equal
    - op: add
      path: "/spec/template/spec/nodeSelector/kubernetes.io~1hostname"
      value: cluster-control-plane
  target:
    kind: Deployment
    name: metrics-server
    namespace: kube-system
resources:
- https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
EOF

$ kubectl apply --kustomize=.
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created

$ kubectl wait pods --for=condition=ready \
--namespace=kube-system --selector=k8s-app=metrics-server
pod/metrics-server-7dbd8b5cc9-brs2z condition met

$ rm --verbose kustomization.yaml
removed 'kustomization.yaml'
```

```bash
$ cat >kustomization.yaml <<EOF
patches:
- patch: |-
    - op: add
      path: "/spec/template/spec/nodeSelector/kubernetes.io~1hostname"
      value: cluster-control-plane
  target:
    kind: Deployment
    name: ingress-nginx-controller
    namespace: ingress-nginx
resources:
- https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
EOF

$ kubectl apply --kustomize=.
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created

$ kubectl wait pods --for=condition=ready \
--namespace=ingress-nginx --selector=app.kubernetes.io/component=controller --timeout=60s
pod/ingress-nginx-controller-5db8bc667-2qvdj condition met

$ rm --verbose kustomization.yaml
removed 'kustomization.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
