# DRAFT: EXPERIMENT KUBERNETES

## REFERENCES

https://www.youtube.com/watch?v=zXCFyKc1_H4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #19 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.affinity.nodeAffinity | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: nodeAffinity <NodeAffinity>

DESCRIPTION:
    Describes node affinity scheduling rules for the pod.
    Node affinity is a group of node affinity scheduling rules.

FIELDS:
  preferredDuringSchedulingIgnoredDuringExecution	<[]PreferredSchedulingTerm>
    The scheduler will prefer to schedule pods to nodes that satisfy the
    affinity expressions specified by this field, but it may choose a node that
    violates one or more of the expressions. The node that is most preferred is
    the one with the greatest sum of weights, i.e. for each node that meets all
    of the scheduling requirements (resource request, requiredDuringScheduling
    affinity expressions, etc.), compute a sum by iterating through the elements
    of this field and adding "weight" to the sum if the node matches the
    corresponding matchExpressions; the node(s) with the highest sum are the
    most preferred.
|...|

$ kubectl explain pods.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution \
--recursive | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: preferredDuringSchedulingIgnoredDuringExecution <[]PreferredSchedulingTerm>

DESCRIPTION:
    The scheduler will prefer to schedule pods to nodes that satisfy the
    affinity expressions specified by this field, but it may choose a node that
    violates one or more of the expressions. The node that is most preferred is
    the one with the greatest sum of weights, i.e. for each node that meets all
    of the scheduling requirements (resource request, requiredDuringScheduling
    affinity expressions, etc.), compute a sum by iterating through the elements
    of this field and adding "weight" to the sum if the node matches the
    corresponding matchExpressions; the node(s) with the highest sum are the
    most preferred.
    An empty preferred scheduling term matches all objects with implicit weight
    0 (i.e. itʼs a no-op). A null preferred scheduling term matches no objects
    (i.e. is also a no-op).

FIELDS:
  preference	<NodeSelectorTerm> -required-
    matchExpressions	<[]NodeSelectorRequirement>
      key	<string> -required-
      operator	<string> -required-
      enum: DoesNotExist, Exists, Gt, In, ....
      values	<[]string>
    matchFields	<[]NodeSelectorRequirement>
      key	<string> -required-
      operator	<string> -required-
      enum: DoesNotExist, Exists, Gt, In, ....
      values	<[]string>
  weight	<integer> -required-
```
