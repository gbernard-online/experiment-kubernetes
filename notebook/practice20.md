# WIP: EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=zXCFyKc1_H4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=e4dpaiIEltk&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #20 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity.podAffinity | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: podAffinity <PodAffinity>

DESCRIPTION:
    Describes pod affinity scheduling rules (e.g. co-locate this pod in the same
    node, zone, etc. as some other pod(s)).
    Pod affinity is a group of inter pod affinity scheduling rules.

FIELDS:
  preferredDuringSchedulingIgnoredDuringExecution	<[]WeightedPodAffinityTerm>
    The scheduler will prefer to schedule pods to nodes that satisfy the
    affinity expressions specified by this field, but it may choose a node that
    violates one or more of the expressions. The node that is most preferred is
    the one with the greatest sum of weights, i.e. for each node that meets all
    of the scheduling requirements (resource request, requiredDuringScheduling
    affinity expressions, etc.), compute a sum by iterating through the elements
    of this field and adding "weight" to the sum if the node has pods which
    matches the corresponding podAffinityTerm; the node(s) with the highest sum
    are the most preferred.

  requiredDuringSchedulingIgnoredDuringExecution	<[]PodAffinityTerm>
    If the affinity requirements specified by this field are not met at
    scheduling time, the pod will not be scheduled onto the node. If the
    affinity requirements specified by this field cease to be met at some point
    during pod execution (e.g. due to a pod label update), the system may or may
    not try to eventually evict the pod from its node. When there are multiple
    elements, the lists of nodes corresponding to each podAffinityTerm are
    intersected, i.e. all terms must be satisfied.

$ kubectl explain pods.spec.affinity.podAntiAffinity | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: podAntiAffinity <PodAntiAffinity>

DESCRIPTION:
    Describes pod anti-affinity scheduling rules (e.g. avoid putting this pod in
    the same node, zone, etc. as some other pod(s)).
    Pod anti affinity is a group of inter pod anti affinity scheduling rules.

FIELDS:
  preferredDuringSchedulingIgnoredDuringExecution	<[]WeightedPodAffinityTerm>
    The scheduler will prefer to schedule pods to nodes that satisfy the
    anti-affinity expressions specified by this field, but it may choose a node
    that violates one or more of the expressions. The node that is most
    preferred is the one with the greatest sum of weights, i.e. for each node
    that meets all of the scheduling requirements (resource request,
    requiredDuringScheduling anti-affinity expressions, etc.), compute a sum by
    iterating through the elements of this field and subtracting "weight" from
    the sum if the node has pods which matches the corresponding
    podAffinityTerm; the node(s) with the highest sum are the most preferred.

  requiredDuringSchedulingIgnoredDuringExecution	<[]PodAffinityTerm>
    If the anti-affinity requirements specified by this field are not met at
    scheduling time, the pod will not be scheduled onto the node. If the
    anti-affinity requirements specified by this field cease to be met at some
    point during pod execution (e.g. due to a pod label update), the system may
    or may not try to eventually evict the pod from its node. When there are
    multiple elements, the lists of nodes corresponding to each podAffinityTerm
    are intersected, i.e. all terms must be satisfied.
```
