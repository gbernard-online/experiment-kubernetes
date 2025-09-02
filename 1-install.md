[![Avatar](img/avatar.webp "ghislain.bernard@gmail.com")](mailto:ghislain.bernard@gmail.com) [![Github](img/github.webp "ghislain-bernard")](https://github.com/ghislain-bernard) [![LinkedIN](img/linkedin.webp "ghislain-bernard")](https://www.linkedin.com/in/ghislain-bernard)

`c[_] 2025`


## INSTALL - KUBECTL - UBUNTU 24


[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io) [![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

REF: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux


```bash
$ kubectl version --client
Client Version: v1.34.0
Kustomize Version: v5.7.1

```

REF: https://github.com/itaysk/kubectl-neat
```bash
$ kubectl-neat version
kubectl-neat version: 2.0.4

```


REF: https://github.com/yannh/kubeconform
```bash
$ kubeconform -v
v0.7.0

```

## INSTALL - KIND - UBUNTU 24

[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io) [![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

REF: https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries

```bash
$ kind --version
kind version 0.29.0

$ kind create cluster --config=kind/single.yaml
Creating cluster "single" ...
 ✓ Ensuring node image (kindest/node:v1.33.1) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-single"
You can now use your cluster with:

kubectl cluster-info --context kind-single

Thanks for using kind! 😊

$ kubectl cluster-info --context kind-single
Kubernetes control plane is running at https://127.0.0.1:41133
CoreDNS is running at https://127.0.0.1:41133/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes
NAME                   STATUS   ROLES           AGE   VERSION
single-control-plane   Ready    control-plane   42s   v1.33.1

```

```bash
$ kubectl apply --filename=https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
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

$ sleep 60

$ kubectl apply --kustomize=kind
serviceaccount/metrics-server unchanged
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/system:metrics-server unchanged
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader unchanged
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server unchanged
service/metrics-server unchanged
deployment.apps/metrics-server configured
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io unchanged

$ sleep 30

```

## INSTALL - KIND - CLUSTER - UBUNTU 24

[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io) [![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

```bash
$ kind --version
kind version 0.29.0

$ kind create cluster --config=kind/cluster.yaml
Creating cluster "cluster" ...
 ✓ Ensuring node image (kindest/node:v1.33.1) 🖼
 ✓ Preparing nodes 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-cluster

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

$ kubectl cluster-info --context kind-cluster
Kubernetes control plane is running at https://127.0.0.1:43553
CoreDNS is running at https://127.0.0.1:43553/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
cluster-control-plane   Ready    control-plane   38s   v1.33.1
cluster-worker          Ready    <none>          23s   v1.33.1

```

```bash
$ kubectl apply --filename=https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
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

$ sleep 60

$ kubectl apply --kustomize=kind
serviceaccount/metrics-server unchanged
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/system:metrics-server unchanged
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader unchanged
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server unchanged
service/metrics-server unchanged
deployment.apps/metrics-server configured
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io unchanged

$ sleep 30

```

## INSTALL - KIND - CLUSTER-COLOR - UBUNTU 24

[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io) [![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

```bash
$ kind --version
kind version 0.29.0

$ kind create cluster --config=kind/cluster-color.yaml
Creating cluster "cluster-color" ...
 ✓ Ensuring node image (kindest/node:v1.33.1) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl conll
text to "kind-cluster-color"
You can now use your cluster with:

kubectl cluster-info --context kind-cluster-color

Have a nice day! 👋

$ kubectl cluster-info --context kind-cluster-color
Kubernetes control plane is running at https://127.0.0.1:33463
CoreDNS is running at https://127.0.0.1:33463/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes
NAME                          STATUS   ROLES           AGE   VERSION
cluster-color-control-plane   Ready    control-plane   47s   v1.33.1
cluster-color-worker-green    Ready    <none>          37s   v1.33.1
cluster-color-worker-orange   Ready    <none>          37s   v1.33.1
cluster-color-worker-red      Ready    <none>          37s   v1.33.1

```

```bash
$ kubectl apply --filename=https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
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

$ sleep 60

$ kubectl apply --kustomize=kind
serviceaccount/metrics-server unchanged
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/system:metrics-server unchanged
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader unchanged
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server unchanged
service/metrics-server unchanged
deployment.apps/metrics-server configured
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io unchanged

$ sleep 30

```

## INSTALL - MICROK8S - UBUNTU 24

[![MikroK8s](img/mikrok8s.webp "MikroK8s")](https://microk8s.io) [![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

```bash
$ sudo snap install --classic microk8s
...
microk8s (1.32/stable) v1.32.3 from Canonical✓ installed

$ sudo microk8s version
MicroK8s v1.32.3 revision 8148

$ sudo usermod --append --groups=microk8s user

$ logout

```

```bash
$ sudo microk8s status --wait-ready
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

$ sudo microk8s kubectl get nodes
NAME     STATUS   ROLES    AGE    VERSION
ubuntu   Ready    <none>   3m9s   v1.32.3

$ sudo microk8s kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   3m20s

$ mkdir --parents --verbose $HOME/.kube
mkdir: created directory '/home/user/.kube'

$ sudo microk8s kubectl config view --raw | tee $HOME/.kube/config
...

$ kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
ubuntu   Ready    <none>   7m22s   v1.32.3

$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   7m38s

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

$ sleep 60

$ sudo microk8s enable metrics-server
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

## INSTALL - NFS - UBUNTU 24

[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

```bash
$ apt-cache policy nfs-kernel-server
nfs-kernel-server:
  Installed: (none)
  Candidate: 1:2.6.4-3ubuntu5.1
  Version table:
     1:2.6.4-3ubuntu5.1 500
        500 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1:2.6.4-3ubuntu5 500
        500 http://archive.ubuntu.com/ubuntu noble/main amd64 Packages

$ sudo apt-get install --yes nfs-kernel-server
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
...
Created symlink /etc/systemd/system/nfs-mountd.service.requires/fsidd.service → /usr/lib/systemd/system/fsidd.service.
Created symlink /etc/systemd/system/nfs-server.service.requires/fsidd.service → /usr/lib/systemd/system/fsidd.service.
Created symlink /etc/systemd/system/nfs-client.target.wants/nfs-blkmap.service → /usr/lib/systemd/system/nfs-blkmap.service.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
nfs-mountd.service is a disabled or a static unit, not starting it.
nfsdcld.service is a disabled or a static unit, not starting it.

Creating config file /etc/exports with new version

Creating config file /etc/default/nfs-kernel-server with new version
...

$ sudo mkdir --verbose /srv/nfs
mkdir: created directory '/srv/nfs'

$ sudo chmod --verbose g-rx,o-rx /srv/nfs
mode of '/srv/nfs' changed from 0755 (rwxr-xr-x) to 0700 (rwx------)

$ echo '/srv/nfs *(fsid=root,insecure,no_root_squash,no_subtree_check,rw,sync)' | sudo tee --append /etc/exports
/srv/nfs *(fsid=root,insecure,no_root_squash,no_subtree_check,rw,sync)

$ sudo systemctl restart nfs-server rpcbind

$ sudo exportfs -av
exporting *:/srv/nfs

$ systemctl is-active nfs-server.service
active

$ systemctl is-active rpcbind.service
active

```

`-EOF-`

[![Monster](img/monster.webp)](mailto:ghislain.bernard@gmail.com)
