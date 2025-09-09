[![Gmail](avatar.webp "ghislain.bernard@gmail.com")](mailto:ghislain.bernard@gmail.com) [![Github](github.webp "ghislain-bernard")](https://github.com/ghislain-bernard) [![LinkedIN](linkedin.webp "ghislain-bernard")](https://www.linkedin.com/in/ghislain-bernard)

## PRACTICE - HELM - KIND - UBUNTU 24

[![Helm](img/helm.webp "Helm")](https://helm.sh) [![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io) [![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

```bash
$ helm upgrade nginx nginx-helm --create-namespace --dry-run --install --namespace=nginx --values=nginx-helm/values-overrides.yaml
Release "nginx" does not exist. Installing it now.
...
NOTES:
revision #1 installed

$ helm upgrade nginx nginx-helm --create-namespace --install --namespace=nginx --values=nginx-helm/values-overrides.yaml
Release "nginx" does not exist. Installing it now.
NAME: nginx
LAST DEPLOYED: Mon Jun 16 17:44:27 2025
NAMESPACE: nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
revision #1 installed

$ helm history nginx --namespace=nginx
REVISION        UPDATED                         STATUS          CHART                   APP VERSION     DESCRIPTION
1               Mon Jun 16 17:44:27 2025        deployed        nginx-helm-0.0.1        1.27.4          Install complete

$ sleep 30

$ curl --cacert nginx-helm/files/ssl/CA.crt --fail --ipv4 --location --show-error --silent https://localhost
nginx nginx nginx

$ helm uninstall nginx --namespace=nginx
release "nginx" uninstalled

$ kubectl rollout restart deployment ingress-nginx-controller --namespace=ingress-nginx
deployment.apps/ingress-nginx-controller restarted

```

## PRACTICE - HELM - MICROK8S - UBUNTU 24

[![Helm](img/helm.webp "Helm")](https://helm.sh) [![MikroK8s](img/mikrok8s.webp "MikroK8s")](https://microk8s.io) [![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com/)

```bash
$ helm upgrade nginx nginx-helm --create-namespace --install --namespace=nginx --values=nginx-helm/values-overrides.yaml
Release "nginx" does not exist. Installing it now.
NAME: nginx
LAST DEPLOYED: Wed Jun 18 14:58:32 2025
NAMESPACE: nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
revision #1 installed

$ sleep 30

$ curl --cacert nginx-helm/files/ssl/CA.crt --fail --ipv4 --location --show-error --silent https://localhost
nginx nginx nginx

$ helm uninstall --namespace=nginx nginx
release "nginx" uninstalled

$ kubectl rollout restart daemonset nginx-ingress-microk8s-controller --namespace=ingress
daemonset.apps/nginx-ingress-microk8s-controller restarted

```
