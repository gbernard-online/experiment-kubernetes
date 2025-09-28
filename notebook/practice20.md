# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=A0Q04eIg1kA&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #20 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain podʼs.spec.affinity --output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: affinity <Object>

DESCRIPTION:
     If specified, the pods scheduling constraints

     Affinity is a group of affinity scheduling rules.

FIELDS:
|...|

   podAffinity	<Object>
     Describes pod affinity scheduling rules (e.g. co-locate this pod in the
     same node, zone, etc. as some other pod(s)).
|...|

$ kubectl explain pods.spec.affinity.podAffinity --output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: podAffinity <Object>

DESCRIPTION:
     Describes pod affinity scheduling rules (e.g. co-locate this pod in the
     same node, zone, etc. as some other pod(s)).

     Pod affinity is a group of inter pod affinity scheduling rules.

FIELDS:
|...|

   requiredDuringSchedulingIgnoredDuringExecution	<[]Object>
     If the affinity requirements specified by this field are not met at
     scheduling time, the pod will not be scheduled onto the node. If the
     affinity requirements specified by this field cease to be met at some point
     during pod execution (e.g. due to a pod label update), the system may or
     may not try to eventually evict the pod from its node. When there are
     multiple elements, the lists of nodes corresponding to each podAffinityTerm
     are intersected, i.e. all terms must be satisfied.

$ kubectl explain pods.spec.affinity.podAntiAffinity.requiredDuringSchedulingIgnoredDuringExecution \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: requiredDuringSchedulingIgnoredDuringExecution <[]Object>

DESCRIPTION:
     If the anti-affinity requirements specified by this field are not met at
     scheduling time, the pod will not be scheduled onto the node. If the
     anti-affinity requirements specified by this field cease to be met at some
     point during pod execution (e.g. due to a pod label update), the system may
     or may not try to eventually evict the pod from its node. When there are
     multiple elements, the lists of nodes corresponding to each podAffinityTerm
     are intersected, i.e. all terms must be satisfied.

     Defines a set of pods (namely those matching the labelSelector relative to
     the given namespace(s)) that this pod should be co-located (affinity) or
     not co-located (anti-affinity) with, where co-located is defined as running
     on a node whose value of the label with key <topologyKey> matches that of
     any node on which a pod of the set of pods is running

FIELDS:
   labelSelector	<Object>
     A label query over a set of resources, in this case pods. If itʼs null,
     this PodAffinityTerm matches with no Pods.
|...|

   topologyKey	<string> -required-
     This pod should be co-located (affinity) or not co-located (anti-affinity)
     with the pods matching the labelSelector in the specified namespaces, where
     co-located is defined as running on a node whose value of the label with
     key topologyKey matches that of any node on which any of the selected pods
     is running. Empty topologyKey is not allowed.

$ kubectl explain \
pods.spec.affinity.podAntiAffinity.requiredDuringSchedulingIgnoredDuringExecution.labelSelector \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: labelSelector <Object>

DESCRIPTION:
     A label query over a set of resources, in this case pods. If itʼs null,
     this PodAffinityTerm matches with no Pods.

     A label selector is a label query over a set of resources. The result of
     matchLabels and matchExpressions are ANDed. An empty label selector matches
     all objects. A null label selector matches no objects.

FIELDS:
|...|

   matchLabels	<map[string]string>
     matchLabels is a map of {key,value} pairs. A single {key,value} in the
     matchLabels map is equivalent to an element of matchExpressions, whose key
     field is "key", the operator is "In", and the values array contains only
     "value". The requirements are ANDed.
```
