# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/workloads/pods  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_explain  
https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md  
https://kubernetes.io/docs/concepts/overview/working-with-objects/labels  
https://kubernetes.io/docs/concepts/overview/working-with-objects/names  
https://kubernetes.io/docs/concepts/configuration/manage-resources-containers  
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs  
https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top

https://www.youtube.com/watch?v=ElVwAByDkf4&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=MbQllww-qRw&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=3iyMOfgZqAo&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=naLtWR6uEWY&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #2 - KUBERNETES - MICROK8S - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![MicroK8s](img/microk8s.webp "MikroK8s")](https://microk8s.io)1
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl run nginx --dry-run=client --image=nginx:alpine --output=yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

$ kubectl explain pods | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
  apiVersion	<string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind	<string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata	<ObjectMeta>
    Standard objectʼs metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec	<PodSpec>
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status	<PodStatus>
    Most recently observed status of the pod. This data may not be up to date.
    Populated by the system. Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

$ kubectl explain pods.metadata | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: metadata <ObjectMeta>

DESCRIPTION:
    Standard objectʼs metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
    ObjectMeta is metadata that all persisted resources must have, which
    includes all objects users must create.

FIELDS:
|...|

  creationTimestamp	<string>
    CreationTimestamp is a timestamp representing the server time when this
    object was created. It is not guaranteed to be set in happens-before order
    across separate operations. Clients may not set this value. It is
    represented in RFC3339 form and is in UTC.

    Populated by the system. Read-only. Null for lists. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
|...|

  labels	<map[string]string>
    Map of string keys and values that can be used to organize and categorize
    (scope and select) objects. May match selectors of replication controllers
    and services. More info:
    https://kubernetes.io/docs/concepts/overview/working-with-objects/labels
|...|

  name	<string>
    Name must be unique within a namespace. Is required when creating resources,
    although some resources may allow a client to request the generation of an
    appropriate name automatically. Name is primarily intended for creation
    idempotence and configuration definition. Cannot be updated. More info:
    https://kubernetes.io/docs/concepts/overview/working-with-objects/names#names
|...|

$ kubectl explain pods.spec | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: spec <PodSpec>

DESCRIPTION:
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
    PodSpec is a description of a pod.

FIELDS:
|...|

  containers	<[]Container> -required-
    List of containers belonging to the pod. Containers cannot currently be
    added or removed. There must be at least one container in a Pod. Cannot be
    updated.
|...|

  restartPolicy	<string>
  enum: Always, Never, OnFailure
    Restart policy for all containers within the pod. One of Always, OnFailure,
    Never. In some contexts, only a subset of those values may be permitted.
    Default to Always. More info:
    https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy

    Possible enum values:
     - `"Always"`
     - `"Never"`
     - `"OnFailure"`
|...|

$ kubectl explain pods.spec.containers | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: containers <[]Container>

DESCRIPTION:
    List of containers belonging to the pod. Containers cannot currently be
    added or removed. There must be at least one container in a Pod. Cannot be
    updated.
    A single application container that you want to run within a pod.

FIELDS:
|...|

  image	<string>
    Container image name. More info:
    https://kubernetes.io/docs/concepts/containers/images This field is optional
    to allow higher level config management to default or override container
    images in workload controllers like Deployments and StatefulSets.
|...|

  name	<string> -required-
    Name of the container specified as a DNS_LABEL. Each container in a pod must
    have a unique name (DNS_LABEL). Cannot be updated.
|...|

  resources	<ResourceRequirements>
    Compute Resources required by this container. Cannot be updated. More info:
    https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
|...|

$ kubectl explain pods.status | cat --squeeze-blank
KIND:       Pod
VERSION:    v1

FIELD: status <PodStatus>

DESCRIPTION:
    Most recently observed status of the pod. This data may not be up to date.
    Populated by the system. Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
    PodStatus represents information about the status of a pod. Status may trail
    the actual state of a system, especially if the node that hosts the pod
    cannot contact the control plane.

FIELDS:
|...|

  phase	<string>
  enum: Failed, Pending, Running, Succeeded, ....
    The phase of a Pod is a simple, high-level summary of where the Pod is in
    its lifecycle. The conditions array, the reason and message fields, and the
    individual container status arrays contain more detail about the podʼs
    status. There are five possible phase values:

    Pending: The pod has been accepted by the Kubernetes system, but one or more
    of the container images has not been created. This includes time before
    being scheduled as well as time spent downloading images over the network,
    which could take a while. Running: The pod has been bound to a node, and all
    of the containers have been created. At least one container is still
    running, or is in the process of starting or restarting. Succeeded: All
    containers in the pod have terminated in success, and will not be restarted.
    Failed: All containers in the pod have terminated, and at least one
    container has terminated in failure. The container either exited with
    non-zero status or was terminated by the system. Unknown: For some reason
    the state of the pod could not be obtained, typically due to an error in
    communicating with the host of the pod.

    More info:
    https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-phase

    Possible enum values:
     - `"Failed"` means that all containers in the pod have terminated, and at
    least one container has terminated in a failure (exited with a non-zero exit
    code or was stopped by the system).
     - `"Pending"` means the pod has been accepted by the system, but one or
    more of the containers has not been started. This includes time before being
    bound to a node, as well as time spent pulling images onto the host.
     - `"Running"` means the pod has been bound to a node and all of the
    containers have been started. At least one container is still running or is
    in the process of being restarted.
     - `"Succeeded"` means that all containers in the pod have voluntarily
    terminated with a container exit code of 0, and the system is not going to
    restart any of these containers.
     - `"Unknown"` means that for some reason the state of the pod could not be
    obtained, typically due to an error in communicating with the host of the
    pod. Deprecated: It isnʼt being set since 2015
    (74da3b14b0c0f658b3bb8d2def5094686d0e9095)
|...|
```

```bash
$ kubectl run nginx --image=nginx:alpine
nginx

$ kubectl get pods --output=name
pod/nginx

$ kubectl logs pod/nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
|...|

$ kubectl get events --for=pod/nginx
LAST SEEN   TYPE     REASON      OBJECT      MESSAGE
48s         Normal   Scheduled   pod/nginx   Successfully assigned default/nginx to ubuntu
47s         Normal   Pulled      pod/nginx   Container image "nginx:alpine" already present on machine
47s         Normal   Created     pod/nginx   Created container: nginx
46s         Normal   Started     pod/nginx   Started container nginx

$ kubectl get events --for=pod/nginx --watch
LAST SEEN   TYPE     REASON      OBJECT      MESSAGE
48s         Normal   Scheduled   pod/nginx   Successfully assigned default/nginx to ubuntu
47s         Normal   Pulled      pod/nginx   Container image "nginx:alpine" already present on machine
47s         Normal   Created     pod/nginx   Created container: nginx
46s         Normal   Started     pod/nginx   Started container nginx
^C

$ kubectl top node
NAME     CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
ubuntu   337m         8%       1618Mi          42%

$ kubectl top pod
NAME    CPU(cores)   MEMORY(bytes)
nginx   0m           4Mi

$ kubectl delete pod/nginx
pod "nginx" deleted from default namespace
```

```bash
$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure -- sleep 10
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - sleep
    - "10"
    image: alpine:latest
    name: alpine
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: OnFailure
status: {}

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --restart=OnFailure -- sleep 10 |
kubectl-neat | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: alpine
  name: alpine
spec:
  containers:
  - args:
    - sleep
    - "10"
    image: alpine:latest
    name: alpine
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl apply --filename=pod.yaml
pod/alpine unchanged

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          9s

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          18s

$ kubectl get pods alpine --output=yaml
apiVersion: v1
kind: Pod
metadata:
|...|

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
