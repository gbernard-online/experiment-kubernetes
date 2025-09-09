[![Gmail](avatar.webp "ghislain.bernard@gmail.com")](mailto:ghislain.bernard@gmail.com) [![Github](github.webp "ghislain-bernard")](https://github.com/ghislain-bernard) [![LinkedIN](linkedin.webp "ghislain-bernard")](https://www.linkedin.com/in/ghislain-bernard)

## PURGE - KIND - UBUNTU 24

```bash
$ kind delete clusters --all
Deleted nodes: ["single-control-plane"]
Deleted clusters: ["single"]

$ rm --verbose $HOME/.kube/config
removed '/home/user/.kube/config'

$ rm --force --recursive $HOME/.kube/cache

```

## PURGE - KIND - CLUSTER - UBUNTU 24

```bash
$ kind delete clusters --all
Deleted nodes: ["cluster-control-plane" "cluster-worker"]
Deleted clusters: ["cluster"]

$ rm --verbose $HOME/.kube/config
removed '/home/user/.kube/config'

$ rm --force --recursive $HOME/.kube/cache

```

## PURGE - KIND - CLUSTER-COLOR - UBUNTU 24

```bash
$ kind delete clusters --all
Deleted nodes: ["cluster-color-worker" "cluster-color-worker2" "cluster-color-control-plane" "cluster-color-worker3"]
Deleted clusters: ["cluster-color"]

$ rm --verbose $HOME/.kube/config
removed '/home/user/.kube/config'

$ rm --force --recursive $HOME/.kube/cache

```

## PURGE - MICROK8S - UBUNTU 24

```bash
$ sudo snap remove --purge microk8s
microk8s removed

$ rm --verbose $HOME/.kube/config
removed '/home/user/.kube/config'

$ rm --force --recursive $HOME/.kube/cache

```
