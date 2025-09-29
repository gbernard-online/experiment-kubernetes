# EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=e4dpaiIEltk&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #20 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ $ kubectl explain pods.spec.topologySpreadConstraints --output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: topologySpreadConstraints <[]Object>

DESCRIPTION:
     TopologySpreadConstraints describes how a group of pods ought to spread
     across topology domains. Scheduler will schedule pods in a way which abides
     by the constraints. All topologySpreadConstraints are ANDed.

     TopologySpreadConstraint specifies how to spread matching pods among the
     given topology.

FIELDS:
   labelSelector	<Object>
     LabelSelector is used to find matching pods. Pods that match this label
     selector are counted to determine the number of pods in their corresponding
     topology domain.
|...|

   maxSkew	<integer> -required-
     MaxSkew describes the degree to which pods may be unevenly distributed.
     When `whenUnsatisfiable=DoNotSchedule`, it is the maximum permitted
     difference between the number of matching pods in the target topology and
     the global minimum. The global minimum is the minimum number of matching
     pods in an eligible domain or zero if the number of eligible domains is
     less than MinDomains. For example, in a 3-zone cluster, MaxSkew is set to
     1, and pods with the same labelSelector spread as 2/2/1: In this case, the
     global minimum is 1. | zone1 | zone2 | zone3 | | P P | P P | P | - if
     MaxSkew is 1, incoming pod can only be scheduled to zone3 to become 2/2/2;
     scheduling it onto zone1(zone2) would make the ActualSkew(3-1) on
     zone1(zone2) violate MaxSkew(1). - if MaxSkew is 2, incoming pod can be
     scheduled onto any zone. When `whenUnsatisfiable=ScheduleAnyway`, it is
     used to give higher precedence to topologies that satisfy it. Itʼss a
     required field. Default value is 1 and 0 is not allowed.
|...|

   topologyKey	<string> -required-
     TopologyKey is the key of node labels. Nodes that have a label with this
     key and identical values are considered to be in the same topology. We
     consider each <key, value> as a "bucket", and try to put balanced number of
     pods into each bucket. We define a domain as a particular instance of a
     topology. Also, we define an eligible domain as a domain whose nodes meet
     the requirements of nodeAffinityPolicy and nodeTaintsPolicy. e.g. If
     TopologyKey is "kubernetes.io/hostname", each Node is a domain of that
     topology. And, if TopologyKey is "topology.kubernetes.io/zone", each zone
     is a domain of that topology. It's a required field.

   whenUnsatisfiable	<string> -required-
     WhenUnsatisfiable indicates how to deal with a pod if it doesn't satisfy
     the spread constraint. - DoNotSchedule (default) tells the scheduler not to
     schedule it. - ScheduleAnyway tells the scheduler to schedule the pod in
     any location, but giving higher precedence to topologies that would help
     reduce the skew. A constraint is considered "Unsatisfiable" for an incoming
     pod if and only if every possible node assignment for that pod would
     violate "MaxSkew" on some topology. For example, in a 3-zone cluster,
     MaxSkew is set to 1, and pods with the same labelSelector spread as 3/1/1:
     | zone1 | zone2 | zone3 | | P P P | P | P | If WhenUnsatisfiable is set to
     DoNotSchedule, incoming pod can only be scheduled to zone2(zone3) to become
     3/2/1(3/1/2) as ActualSkew(2-1) on zone2(zone3) satisfies MaxSkew(1). In
     other words, the cluster can still be imbalanced, but scheduler won't make
     it *more* imbalanced. It's a required field.

     Possible enum values:
     - `"DoNotSchedule"` instructs the scheduler not to schedule the pod when
     constraints are not satisfied.
     - `"ScheduleAnyway"` instructs the scheduler to schedule the pod even if
     constraints are not satisfied.
```

```bash
$ kubectl create deployment nginx --dry-run=client --image=nginx:alpine --output=json --replicas=18 |
jq '.spec.template.spec.topologySpreadConstraints = input' - <(echo '[
  {
    "labelSelector": {
      "matchLabels": {
        "app": "nginx"
      }
    },
    "maxSkew": 1,
    "topologyKey": "color",
    "whenUnsatisfiable": "DoNotSchedule"
  }
]') | kubectl-neat --output=yaml  | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 18
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
      topologySpreadConstraints:
      - labelSelector:
          matchLabels:
            app: nginx
        maxSkew: 1
        topologyKey: color
        whenUnsatisfiable: DoNotSchedule

$ kubeconform -verbose deployment.yaml
deployment.yaml - Deployment nginx is valid

$ kubectl apply --filename=deployment.yaml
deployment.apps/nginx created

$ kubectl annotate deployments.apps nginx kubernetes.io/change-cause=nginx:alpine
deployment.apps/nginx annotated

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine:redis

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          19s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-green
cluster-worker-red

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      6 cluster-worker-green
      6 cluster-worker-red
      6 cluster-worker-yellow

$ kubectl get pods --output=yaml --selector=app=nginx |
yq '.items[] | select(.spec.nodeName=="cluster-worker-red").metadata.name' |
xargs --no-run-if-empty kubectl delete pods
pod "nginx-7cbb9b66db-4dt4d" deleted
pod "nginx-7cbb9b66db-d48pg" deleted
pod "nginx-7cbb9b66db-ljrsr" deleted
pod "nginx-7cbb9b66db-mqvjz" deleted
pod "nginx-7cbb9b66db-rrlzb" deleted
pod "nginx-7cbb9b66db-zb8dd" deleted

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          77s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-green
cluster-worker-green
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-green
cluster-worker-green
cluster-worker-red
cluster-worker-yellow
cluster-worker-green
cluster-worker-red
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-green

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      6 cluster-worker-green
      6 cluster-worker-red
      6 cluster-worker-yellow

$ kubectl patch deployments.apps nginx --patch='[
{
  "op": "replace",
  "path": "/spec/template/spec/topologySpreadConstraints/0/maxSkew",
  "value": 6
}
]' --type=json
deployment.apps/nginx patched

$ kubectl get deployments.apps nginx --output=yaml | yq .spec.template.spec.topologySpreadConstraints
- labelSelector:
    matchLabels:
      app: nginx
  maxSkew: 6
  topologyKey: color
  whenUnsatisfiable: DoNotSchedule

$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         nginx:alpine
2         nginx:alpine

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          2m05s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-red
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-green
cluster-worker-red

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      5 cluster-worker-green
      5 cluster-worker-red
      8 cluster-worker-yellow

$ kubectl get pods --output=yaml --selector=app=nginx |
yq '.items[] | select(.spec.nodeName=="cluster-worker-red").metadata.name' |
xargs --no-run-if-empty kubectl delete pods
pod "nginx-56d479d685-mc2hk" deleted from default namespace
pod "nginx-56d479d685-mwvfh" deleted from default namespace
pod "nginx-56d479d685-ng5xq" deleted from default namespace
pod "nginx-56d479d685-sk4hn" deleted from default namespace
pod "nginx-56d479d685-z57r5" deleted from default namespace

$ kubectl get deployments.apps nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   18/18   18           18          2m45s

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-green
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-green
cluster-worker-yellow
cluster-worker-red
cluster-worker-red
cluster-worker-green
cluster-worker-green
cluster-worker-green

$ kubectl get pods --output=yaml --selector=app=nginx | yq .items[].spec.nodeName | sort | uniq --count
      7 cluster-worker-green
      3 cluster-worker-red
      8 cluster-worker-yellow

$ kubectl delete --filename=deployment.yaml
deployment.apps "nginx" deleted

$ rm --verbose deployment.yaml
removed 'deployment.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
