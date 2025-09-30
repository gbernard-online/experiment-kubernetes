# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/configuration/secret

https://www.youtube.com/watch?v=5b3kkJ0pUjA&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #10 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep secrets
secrets                                        v1                                true    Secret

$ kubectl explain secrets --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl create secret generic alpine --dry-run=client --from-literal=password='~=*SECRET*=~' \
--output=yaml | yq '.type="Opaque"' | kubectl-neat | tee secret.yaml
apiVersion: v1
data:
  password: fj0qU0VDUkVUKj1+
kind: Secret
metadata:
  name: alpine
type: Opaque

$ kubectl apply --filename=secret.yaml
secret/alpine created

$ kubectl get secrets alpine
NAME     TYPE     DATA   AGE
alpine   Opaque   1      10s

$ kubectl describe secrets alpine
Name:         alpine
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  12 bytes

$ kubectl get secrets alpine --output=jsonpath='{.data.password}' && echo
fj0qU0VDUkVUKj1+

$ kubectl get secrets alpine --output=jsonpath='{.data.password}' | base64 --decode && echo
~=*SECRET*=~

$ kubectl run alpine --dry-run=client --image=alpine:latest --output=yaml --overrides='[
{
  "op": "add",
  "path": "/spec/containers/0/env",
  "value": [
    {
      "name": "PASSWORD",
      "valueFrom": {
        "secretKeyRef": {
          "key": "password",
          "name": "alpine"
        }
      }
    }
  ]
}
]' --override-type=json --restart=OnFailure -- sleep 60 | kubectl-neat | tee pod.yaml
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
    - "60"
    env:
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          key: password
          name: alpine
    image: alpine:latest
    name: alpine
  restartPolicy: OnFailure

$ kubectl apply --filename=pod.yaml
pod/alpine created

$ kubectl get pods alpine
NAME     READY   STATUS    RESTARTS   AGE
alpine   1/1     Running   0          7s

$ kubectl exec alpine -- env | fgrep PASSWORD
PASSWORD=~=*SECRET*=~

$ kubectl get pods alpine
NAME     READY   STATUS      RESTARTS   AGE
alpine   0/1     Completed   0          65s

$ kubectl delete --filename=pod.yaml
pod "alpine" deleted

$ kubectl delete --filename=secret.yaml
secret "alpine" deleted

$ rm --verbose secret.yaml pod.yaml
removed 'secret.yaml'
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
