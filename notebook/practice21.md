# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=zXCFyKc1_H4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #20 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity.podAffinity --output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: podAffinity <Object>

DESCRIPTION:
     Describes pod affinity scheduling rules (e.g. co-locate this pod in the
     same node, zone, etc. as some other pod(s)).

     Pod affinity is a group of inter pod affinity scheduling rules.

FIELDS:
   preferredDuringSchedulingIgnoredDuringExecution	<[]Object>
     The scheduler will prefer to schedule pods to nodes that satisfy the
     affinity expressions specified by this field, but it may choose a node that
     violates one or more of the expressions. The node that is most preferred is
     the one with the greatest sum of weights, i.e. for each node that meets all
     of the scheduling requirements (resource request, requiredDuringScheduling
     affinity expressions, etc.), compute a sum by iterating through the
     elements of this field and adding "weight" to the sum if the node has pods
     which matches the corresponding podAffinityTerm; the node(s) with the
     highest sum are the most preferred.
|...

$ kubectl explain pods.spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: preferredDuringSchedulingIgnoredDuringExecution <[]Object>

DESCRIPTION:
     The scheduler will prefer to schedule pods to nodes that satisfy the
     anti-affinity expressions specified by this field, but it may choose a node
     that violates one or more of the expressions. The node that is most
     preferred is the one with the greatest sum of weights, i.e. for each node
     that meets all of the scheduling requirements (resource request,
     requiredDuringScheduling anti-affinity expressions, etc.), compute a sum by
     iterating through the elements of this field and subtracting "weight" from
     the sum if the node has pods which matches the corresponding
     podAffinityTerm; the node(s) with the highest sum are the most preferred.

     The weights of all of the matched WeightedPodAffinityTerm fields are added
     per-node to find the most preferred node(s)

FIELDS:
   podAffinityTerm	<Object> -required-
     Required. A pod affinity term, associated with the corresponding weight.

   weight	<integer> -required-
     weight associated with matching the corresponding podAffinityTerm, in the
     range 1-100.

$ kubectl explain \
pods.spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution.podAffinityTerm \
--output=plaintext-openapiv2
KIND:     Pod
VERSION:  v1

RESOURCE: podAffinityTerm <Object>

DESCRIPTION:
     Required. A pod affinity term, associated with the corresponding weight.

     Defines a set of pods (namely those matching the labelSelector relative to
     the given namespace(s)) that this pod should be co-located (affinity) or
     not co-located (anti-affinity) with, where co-located is defined as running
     on a node whose value of the label with key <topologyKey> matches that of
     any node on which a pod of the set of pods is running

FIELDS:
   labelSelector	<Object>
     A label query over a set of resources, in this case pods. If it's null,
     this PodAffinityTerm matches with no Pods.
|...|

   topologyKey	<string> -required-
     This pod should be co-located (affinity) or not co-located (anti-affinity)
     with the pods matching the labelSelector in the specified namespaces, where
     co-located is defined as running on a node whose value of the label with
     key topologyKey matches that of any node on which any of the selected pods
     is running. Empty topologyKey is not allowed.
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
