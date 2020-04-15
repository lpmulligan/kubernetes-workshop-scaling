# Kubernetes Horizontal Pod Autoscaler (HPA) Memory Scaling Lab

## HPA

Perform the following steps to see Kubernetes HPA in action. 


## Memory Limits

Deploy the pods and service if they're not still running.  Note: everything is labled with tier=frontend
```shell
❯ kubectl apply -f 05-frontend-deployment-svc.yaml 
deployment.apps/frontend configured
service/frontend configured
```

Apply the HPA that will work with memory.
```shell
❯ kubectl apply -f 07-frontend-hpa-v2-api.yaml 
horizontalpodautoscaler.autoscaling/frontend-v2 created
```

Now let's put the pods under some load and see what happens.

First take note of the HPA.  There shouldn't be any load.
```shell
❯ kubectl get hpa frontend-v2
NAME          REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
frontend-v2   Deployment/frontend   5%/50%    2         5         2          33s
```

Startup the load generator
```shell
> kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh

while true; do wget -q -O- http://frontend.default.svc.cluster.local; done
```

After about 30 seconds or so, check the HPA again and if you haven't closed out of a second terminal you can watch the pods spin up.
```shell
shell 1❯ ❯ kubectl get hpa frontend-v2
NAME          REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
frontend-v2   Deployment/frontend   5%/50%    2         5         2          4m37s

shell 2❯ k get pods -w
NAME                        READY   STATUS    RESTARTS   AGE
frontend-54bb48c447-jp87x   1/1     Running   0          59m
frontend-54bb48c447-kbmmw   1/1     Running   0          52m
load-generator              1/1     Running   0          43s
```
Alright, so nothing happened. Why?  Well we didn't put enough load on the pods because the load generator only generates CPU load.  It's also possible to scale on both CPU and memory if need.  You'll need to put something like this in the HPA manifest
```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-v2
spec:
  maxReplicas: 5
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  metrics:
  - resource:
      name: memory
      target:
          type: Utilization
          averageUtilization: 50
    type: Resource
  - resource:
      name: cpu
      target:
          type: Utilization
          averageUtilization: 50
    type: Resource
```

So let's try something different.  Clean everyting up
```shell
❯ k delete -f 07-frontend-hpa-v2-api.yaml 
horizontalpodautoscaler.autoscaling "frontend-v2" deleted

❯ k delete -f 06-frontend-deployment-svc.yaml 
deployment.apps "frontend" deleted
service "frontend" deleted
```

Deploy a new set of pods with setting we can configure to stress the memory.  If you look at the manifest, you'll see we're using a pod with a utility called `stress` baked in.  We'll use this utility to put the pod under memory pressure. 
```yaml
spec:
      containers:
      - name: oom-app
        image: 'polinux/stress'
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: "200M"
            cpu: "200m"
          requests:
            memory: "50M"
            cpu: "50m"
        command:
            - stress
        args:
            - '--vm'
            - '1'
            - '--vm-bytes'
            - 50M
            - '--vm-hang'
            - '1'
```

```shell
❯ k apply -f 07-frontend-deployment-svc-memtest.yaml
deployment.apps/oom-frontend created
service/frontend created
```

This deployment should cause the cluster to scale up from the initial 2 replicas.

```shell
❯ kubectl get hpa frontend-v2
NAME          REFERENCE                 TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
frontend-v2   Deployment/oom-frontend   106%/50%   2         5         5          17m

❯ kubectl describe hpa frontend-v2
...
Metrics:                                                  ( current / target )
  resource memory on pods  (as a percentage of request):  106% (53465907200m) / 50%
Min replicas:                                             2
Max replicas:                                             5
Deployment pods:                                          5 current / 5 desired
...
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  12m   horizontal-pod-autoscaler  New size: 2; reason: Current number of replicas below Spec.MinReplicas
  Normal  SuccessfulRescale  12m   horizontal-pod-autoscaler  New size: 4; reason: memory resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  11m   horizontal-pod-autoscaler  New size: 5; reason:
```

### Clean Up

Clean up everything
```shell
❯ kubectl delete -f 07-frontend-hpa-v2-api.yaml 
horizontalpodautoscaler.autoscaling "frontend-v2" deleted

❯ kubectl delete -f 07-frontend-deployment-svc-memtest.yaml 
deployment.apps "oom-frontend" deleted
service "frontend" deleted
```

Next: [Vertical Scaling](08-verical-scaling.md)
