# EXPERIMENT KUBERNETES

## REFERENCES

https://kubernetes.io/docs/concepts/configuration/manage-resources-containers

https://www.youtube.com/watch?v=tt09ZyoHlvI&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap  
https://www.youtube.com/watch?v=TE3syui3Kk8&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #32 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl explain pods.spec.containers.resources --output=plaintext-openapiv2
|...|

$ kubectl explain pods.spec.containers.resources.limits --output=plaintext-openapiv2
|...|

$ kubectl explain pods.spec.containers.resources.requests --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl run apache --dry-run=client --image=registry.k8s.io/hpa-example --output=json |
kubectl-neat --output=yaml | tee pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: apache
  name: apache
spec:
  containers:
  - image: registry.k8s.io/hpa-example
    name: apach

$ kubeconform -verbose pod.yaml
pod.yaml - Pod apache is valid

$ kubectl apply --filename=pod.yaml
pod/apache created

$ kubectl get pods apache
NAME     READY   STATUS    RESTARTS   AGE
apache   1/1     Running   0          12s

$ kubectl top pod apache
NAME     CPU(cores)   MEMORY(bytes)
apache   1m           8Mi

$ kubectl get pods --output=custom-columns='NAME:.metadata.name,IP:.status.podIP'
NAME     IP
apache   10.244.3.5

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent http://10.244.3.5 && echo
OK!

$ docker run --network=container:cluster-control-plane --rm httpd:latest \
ab -t60 http://10.244.3.5/
This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.244.3.5 (be patient)
Finished 1254 requests

[...]

Concurrency Level:      1
Time taken for tests:   60.047 seconds
Complete requests:      1254
Failed requests:        0
Total transferred:      258324 bytes
HTML transferred:       3762 bytes
Requests per second:    20.88 [#/sec] (mean)
Time per request:       47.884 [ms] (mean)
Time per request:       47.884 [ms] (mean, across all concurrent requests)
Transfer rate:          4.20 [Kbytes/sec] received

[...]

$ kubectl top pod apache
NAME     CPU(cores)   MEMORY(bytes)
apache   995m         12Mi

$ docker run --network=container:cluster-control-plane --rm httpd:latest \
ab -c3 -t60 http://10.244.3.5/
This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.244.3.5 (be patient)
Finished 3648 requests

[...]

Concurrency Level:      3
Time taken for tests:   60.003 seconds
Complete requests:      3648
Failed requests:        0
Total transferred:      751488 bytes
HTML transferred:       10944 bytes
Requests per second:    60.80 [#/sec] (mean)
Time per request:       49.344 [ms] (mean)
Time per request:       16.448 [ms] (mean, across all concurrent requests)
Transfer rate:          12.23 [Kbytes/sec] received

[...]

$ kubectl top pod apache
NAME     CPU(cores)   MEMORY(bytes)
apache   2928m        15Mi

$ kubectl delete --filename=pod.yaml
pod "apache" deleted from default namespace

$ patch --forward --reject-file=- pod.yaml <<EOF
10a11,15
>     resources:
>       limits:
>         cpu: 1000m
>       requests:
>         cpu: 250m
EOF
patching file pod.yaml

$ tail --lines=5 pod.yaml
    resources:
      limits:
        cpu: 1000m
      requests:
        cpu: 250m

$ kubeconform -verbose pod.yaml
pod.yaml - Pod apache is valid

$ kubectl apply --filename=pod.yaml
pod/apache created

$ kubectl get pods apache
NAME     READY   STATUS    RESTARTS   AGE
apache   1/1     Running   0          14s

$ kubectl get pods --output=custom-columns='NAME:.metadata.name,IP:.status.podIP'
NAME     IP
apache   10.244.3.6

$ docker run --network=container:cluster-control-plane --rm httpd:latest \
ab -t60 http://10.244.3.6/
This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.244.3.6 (be patient)
Finished 1247 requests

[...]

Concurrency Level:      1
Time taken for tests:   60.034 seconds
Complete requests:      1247
Failed requests:        0
Total transferred:      256882 bytes
HTML transferred:       3741 bytes
Requests per second:    20.77 [#/sec] (mean)
Time per request:       48.142 [ms] (mean)
Time per request:       48.142 [ms] (mean, across all concurrent requests)
Transfer rate:          4.18 [Kbytes/sec] received

[...]

$ kubectl top pod apache
NAME     CPU(cores)   MEMORY(bytes)
apache   996m         12Mi

$ docker run --network=container:cluster-control-plane --rm httpd:latest \
ab -c3 -t60 http://10.244.3.6/
This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.244.3.6 (be patient)
Finished 1021 requests

[...]

Concurrency Level:      3
Time taken for tests:   60.088 seconds
Complete requests:      1021
Failed requests:        0
Total transferred:      210326 bytes
HTML transferred:       3063 bytes
Requests per second:    16.99 [#/sec] (mean)
Time per request:       176.555 [ms] (mean)
Time per request:       58.852 [ms] (mean, across all concurrent requests)
Transfer rate:          3.42 [Kbytes/sec] received

[...]

$ kubectl top pod apache
NAME     CPU(cores)   MEMORY(bytes)
apache   1002m        14Mi

$ kubectl delete --filename=pod.yaml
pod "apache" deleted from default namespace

$ rm --verbose pod.yaml
removed 'pod.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
