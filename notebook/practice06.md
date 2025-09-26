# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/services-networking/service

https://www.youtube.com/watch?v=Z62WCbIIWyg&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=ZGIfWYnodJQ&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #6 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl create deployment nginx --image=nginx:alpine --replicas=3
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-bmzmc   1/1     Running   0          15s
nginx-7977cdf8f5-kg2l7   1/1     Running   0          15s
nginx-7977cdf8f5-t5kxh   1/1     Running   0          15s

$ kubectl expose deployment nginx --port=8080 --target-port=80
service/nginx exposed

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR
nginx   ClusterIP   10.96.46.237   <none>        8080/TCP   10s   app=nginx

$ curl --connect-timeout 5 --fail --show-error --silent 10.96.46.237:8080
curl: (28) Failed to connect to 10.96.46.237 port 8080 after 5002 ms: Timeout was reached

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.46.237:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
ash -c 'nslookup nginx.default.svc.cluster.local && cat /etc/resolv.conf' | cat --squeeze-blank
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	nginx.default.svc.cluster.local
Address: 10.96.46.237

search default.svc.cluster.local svc.cluster.local cluster.local qemu.lan
nameserver 10.96.0.10
options ndots:5

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx.default.svc.cluster.local:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl run alpine --image=alpine:latest --quiet --restart=Never --rm --stdin --tty -- \
wget -O - -q -T 5 nginx:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl get endpointslices.discovery.k8s.io nginx-p6jj5
NAME          ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
nginx-p6jj5   IPv4          80      10.244.1.28,10.244.2.29,10.244.3.26   95s

$ kubectl scale deployment nginx --replicas=1
deployment.apps/nginx scaled

$ kubectl get pods --selector=app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7977cdf8f5-kg2l7   1/1     Running   0          2m39s

$ kubectl get endpointslices.discovery.k8s.io nginx-p6jj5
NAME          ADDRESSTYPE   PORTS   ENDPOINTS     AGE
nginx-p6jj5   IPv4          80      10.244.3.26   2m10s

$ kubectl delete services nginx
service "nginx" deleted

$ kubectl delete deployments.apps nginx
deployment.apps "nginx" deleted
```

```bash
$ kubectl api-resources --no-headers | fgrep services | fgrep svc
services                            svc        v1                                true    Service

$ kubectl explain services | cat --squeeze-blank
KIND:       Service
VERSION:    v1

DESCRIPTION:
    Service is a named abstraction of software service (for example, mysql)
    consisting of local port (for example 3306) that the proxy listens on, and
    the selector that determines which pods will answer requests sent through
    the proxy.

FIELDS:
|...|

  spec	<ServiceSpec>
    Spec defines the behavior of a service.
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
|...|

$ kubectl explain services.spec | cat --squeeze-blank
KIND:       Service
VERSION:    v1

FIELD: spec <ServiceSpec>

DESCRIPTION:
    Spec defines the behavior of a service.
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
    ServiceSpec describes the attributes that a user creates on a service.

FIELDS:
|...|

  externalName	<string>
    externalName is the external reference that discovery mechanisms will return
    as an alias for this service (e.g. a DNS CNAME record). No proxying will be
    involved.  Must be a lowercase RFC-1123 hostname
    (https://tools.ietf.org/html/rfc1123) and requires `type` to be
    "ExternalName".
|...|

  ports	<[]ServicePort>
    The list of ports that are exposed by this service. More info:
    https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies
|...|

  selector	<map[string]string>
    Route service traffic to pods with label keys and values matching this
    selector. If empty or not present, the service is assumed to have an
    external process managing its endpoints, which Kubernetes will not modify.
    Only applies to types ClusterIP, NodePort, and LoadBalancer. Ignored if type
    is ExternalName. More info:
    https://kubernetes.io/docs/concepts/services-networking/service/
|...|

  type	<string>
  enum: ClusterIP, ExternalName, LoadBalancer, NodePort
    type determines how the Service is exposed. Defaults to ClusterIP. Valid
    options are ExternalName, ClusterIP, NodePort, and LoadBalancer. "ClusterIP"
    allocates a cluster-internal IP address for load-balancing to endpoints.
    Endpoints are determined by the selector or if that is not specified, by
    manual construction of an Endpoints object or EndpointSlice objects. If
    clusterIP is "None", no virtual IP is allocated and the endpoints are
    published as a set of endpoints rather than a virtual IP. "NodePort" builds
    on ClusterIP and allocates a port on every node which routes to the same
    endpoints as the clusterIP. "LoadBalancer" builds on NodePort and creates an
    external load-balancer (if supported in the current cloud) which routes to
    the same endpoints as the clusterIP. "ExternalName" aliases this service to
    the specified externalName. Several other fields do not apply to
    ExternalName services. More info:
    https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types

    Possible enum values:
     - `"ClusterIP"` means a service will only be accessible inside the cluster,
    via the cluster IP.
     - `"ExternalName"` means a service consists of only a reference to an
    external name that kubedns or equivalent will return as a CNAME record, with
    no exposing or proxying of any pods involved.
     - `"LoadBalancer"` means a service will be exposed via an external load
    balancer (if the cloud provider supports it), in addition to 'NodePort'
    type.
     - `"NodePort"` means a service will be exposed on one port of every node,
    in addition to 'ClusterIP' type.

$ kubectl explain services.spec.ports | cat --squeeze-blank
KIND:       Service
VERSION:    v1

FIELD: ports <[]ServicePort>

DESCRIPTION:
    The list of ports that are exposed by this service. More info:
    https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies
    ServicePort contains information on serviceʼs port.

FIELDS:
|...|
  name	<string>
    The name of this port within the service. This must be a DNS_LABEL. All
    ports within a ServiceSpec must have unique names. When considering the
    endpoints for a Service, this must match the 'name' field in the
    EndpointPort. Optional if only one ServicePort is defined on this service.

  port	<integer> -required-
    The port that will be exposed by this service.
|...|
  targetPort	<IntOrString>
    Number or name of the port to access on the pods targeted by the service.
    Number must be in the range 1 to 65535. Name must be an IANA_SVC_NAME. If
    this is a string, it will be looked up as a named port in the target Podʼs
    container ports. If this is not specified, the value of the 'port' field is
    used (an identity map). This field is ignored for services with
    clusterIP=None, and should be omitted or set equal to the 'port' field. More
    info:
    https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=yaml --replicas=3 |
kubectl-neat | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: nginx

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine

$ kubectl create service clusterip nginx --dry-run=client --output=yaml --tcp=8080:80 |
kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - name: 8080-80
    port: 8080
    targetPort: 80
  selector:
    app: nginx

$ kubectl apply --filename=service.yaml
service/ipinfo created

$ kubectl get services nginx --output=wide
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR
nginx   ClusterIP   10.96.179.10   <none>        8080/TCP   15s   app=nginx

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.179.10:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
|...|

$ kubectl delete --filename=service.yaml
service "nginx" deleted

$ rm --verbose service.yaml
removed 'service.yaml'

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

---

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
