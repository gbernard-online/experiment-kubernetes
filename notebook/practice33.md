# EXPERIMENT KUBERNETES

## REFERENCES


https://kubernetes.io/docs/concepts/workloads/autoscaling  
https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale

https://www.youtube.com/watch?v=JIXnwwly2H8&list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap

## PRACTICE #33 - KUBERNETES - KIND - UBUNTU 24

[![Kubernetes](img/kubernetes.webp "Kubernetes")](https://kubernetes.io)1
[![Kind](img/kind.webp "Kind")](https://kind.sigs.k8s.io)0
[![Ubuntu](img/ubuntu.webp "Ubuntu")](https://ubuntu.com)24

```bash
$ kubectl api-resources --no-headers | fgrep horizontalpodautoscalers
|...|

$ kubectl explain horizontalpodautoscaler --output=plaintext-openapiv2
|...|

$ kubectl explain horizontalpodautoscalers.spec --output=plaintext-openapiv2
|...|

$ kubectl explain horizontalpodautoscalers.spec.metrics --output=plaintext-openapiv2
|...|
```

```bash
$ kubectl create deployment apache --dry-run=client --image=registry.k8s.io/hpa-example --output=json | jq '.spec.template.spec.containers[0].resources = input' - <(echo '{
  "limits": {
    "cpu": "1000m"
  },
  "requests": {
    "cpu": "250m"
  }
}') | kubectl-neat --output=yaml | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: apache
  name: apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - image: registry.k8s.io/hpa-example
        name: hpa-example
        resources:
          limits:
            cpu: 1000m
          requests:
            cpu: 250

$ kubeconform -verbose deployment.yaml
deployment.yaml - Deployment apache is valid

$ kubectl apply --filename=deployment.yaml
deployment.apps/apache created

$ kubectl annotate deployments.apps apache kubernetes.io/change-cause=apache:latest
deployment.apps/apache annotated

$ kubectl rollout history deployment apache
deployment.apps/apache
REVISION  CHANGE-CAUSE
1         apache:latest

$ kubectl get deployments.apps apache
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
apache   1/1     1            1           31s

$ kubectl get pods --output=yaml --selector=app=apache | yq .items[].spec.nodeName
cluster-worker-red

$ kubectl expose deployment apache --dry-run=client --port=8080 --target-port=80 --output=yaml |
kubectl-neat | tee service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: apache
  name: apache
spec:
  ports:
  - port: 8080
    targetPort: 80
  selector:
    app: apache

$ kubeconform -verbose service.yaml
service.yaml - Service apache is valid

$ kubectl apply --filename=service.yaml
service/apache exposed

$ kubectl get services apache --output=wide
NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
apache   ClusterIP   10.96.129.108   <none>        8080/TCP   19s   app=apache

$ docker exec cluster-control-plane \
curl --connect-timeout 5 --fail --show-error --silent 10.96.129.108:8080 && echo
OK!

$ docker run --network=container:cluster-control-plane --rm httpd:latest \
ab -c3 -t60 http://10.96.129.108:8080/
This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.96.129.108 (be patient)
Finished 997 requests

|...|

Concurrency Level:      3
Time taken for tests:   60.121 seconds
Complete requests:      997
Failed requests:        0
Total transferred:      205382 bytes
HTML transferred:       2991 bytes
Requests per second:    16.58 [#/sec] (mean)
Time per request:       180.905 [ms] (mean)
Time per request:       60.302 [ms] (mean, across all concurrent requests)
Transfer rate:          3.34 [Kbytes/sec] received

|...|

$ kubectl top pod --selector=app=apache
NAME                      CPU(cores)   MEMORY(bytes)
apache-5b75577d5d-mjzwr   1001m        14Mi

$ kubectl autoscale deployment apache --cpu=500m --dry-run=client --max=9 --output=yaml | 
kubectl-neat | tee horizontalpodautoscaler.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: apache
spec:
  maxReplicas: 9
  metrics:
  - resource:
      name: cpu
      target:
        averageValue: 500m
        type: AverageValue
    type: Resource
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: apache

$ kubeconform -verbose horizontalpodautoscaler.yaml
horizontalpodautoscaler.yaml - HorizontalPodAutoscaler apache is valid

$ kubectl apply --filename=horizontalpodautoscaler.yaml
horizontalpodautoscaler.autoscaling/apache created

$ kubectl get horizontalpodautoscalers.autoscaling apache
NAME     REFERENCE           TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
apache   Deployment/apache   cpu: 1m/500m   1         9         1          19s

$ docker run --network=container:cluster-control-plane --rm httpd:latest \
ab -c3 -t60 http://10.96.129.108:8080/
This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.96.129.108 (be patient)
Finished 1571 requests

|...|

Concurrency Level:      3
Time taken for tests:   60.040 seconds
Complete requests:      1571
Failed requests:        0
Total transferred:      323626 bytes
HTML transferred:       4713 bytes
Requests per second:    26.17 [#/sec] (mean)
Time per request:       114.653 [ms] (mean)
Time per request:       38.218 [ms] (mean, across all concurrent requests)
Transfer rate:          5.26 [Kbytes/sec] received

|...|

$ kubectl get pods --output=yaml --selector=app=apache | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-yellow
cluster-worker-red
cluster-worker-green

$ kubectl top pod --selector=app=apache
NAME                      CPU(cores)   MEMORY(bytes)
apache-5b75577d5d-mjzwr   786m         14Mi
apache-5b75577d5d-qblbx   459m         13Mi
apache-5b75577d5d-rh9ph   230m         11Mi
apache-5b75577d5d-vdl22   270m         12Mi

$ kubectl get horizontalpodautoscalers.autoscaling apache
NAME     REFERENCE           TARGETS          MINPODS   MAXPODS   REPLICAS   AGE
apache   Deployment/apache   cpu: 622m/500m   1         9         4          2m20s

$ docker run --network=container:cluster-control-plane --rm httpd:latest \
ab -c3 -t300 http://10.96.129.108:8080/
This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.96.129.108 (be patient)
Completed 5000 requests
Completed 10000 requests
Finished 14866 requests

|...|

Concurrency Level:      3
Time taken for tests:   300.009 seconds
Complete requests:      14866
Failed requests:        0
Total transferred:      3062396 bytes
HTML transferred:       44598 bytes
Requests per second:    49.55 [#/sec] (mean)
Time per request:       60.543 [ms] (mean)
Time per request:       20.181 [ms] (mean, across all concurrent requests)
Transfer rate:          9.97 [Kbytes/sec] received

|...|

$ kubectl get pods --output=yaml --selector=app=apache | yq .items[].spec.nodeName
cluster-worker-red
cluster-worker-yellow
cluster-worker-yellow
cluster-worker-red
cluster-worker-green

$ kubectl top pod --selector=app=apache
NAME                      CPU(cores)   MEMORY(bytes)
apache-5b75577d5d-mjzwr   575m         14Mi
apache-5b75577d5d-qblbx   450m         13Mi
apache-5b75577d5d-r79nr   539m         13Mi
apache-5b75577d5d-rh9ph   522m         13Mi
apache-5b75577d5d-vdl22   477m         13Mi

$ kubectl get horizontalpodautoscalers.autoscaling apache
NAME     REFERENCE           TARGETS          MINPODS   MAXPODS   REPLICAS   AGE
apache   Deployment/apache   cpu: 491m/500m   1         9         5          7m52s

$ kubectl get horizontalpodautoscalers.autoscaling apache
NAME     REFERENCE           TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
apache   Deployment/apache   cpu: 1m/500m   1         9         5          10m

$ kubectl get horizontalpodautoscalers.autoscaling apache
NAME     REFERENCE           TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
apache   Deployment/apache   cpu: 1m/500m   1         9         1          13m

$ kubectl delete --filename=horizontalpodautoscaler.yaml
kubectl delete --filename=horizontalpodautoscaler.yaml

$ kubectl delete --filename=service.yaml
service "apache" deleted from default namespace

$ kubectl delete --filename=deployment.yaml
deployment.apps "apache" deleted from default namespace

$ rm --verbose deployment.yaml service.yaml horizontalpodautoscaler.yaml
removed 'deployment.yaml'
removed 'service.yaml'
removed 'horizontalpodautoscaler.yaml'
```

&nbsp;

`-`

[![Monster](https://avatars.githubusercontent.com/u/47848582?s=96&v=4 "Boo!")](../README.md)
