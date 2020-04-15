# Kubernetes Horizontal Pod Autoscaler (HPA) CPU Scaling Lab

## HPA

Perform the following steps to see Kubernetes HPA in action. 


## CPU Limits

Deploy the pods and service.  Note: everything is labled with tier=frontend
```shell
❯ kubectl apply -f 06-frontend-deployment-svc.yaml 
deployment.apps/frontend configured
service/frontend configured
```

Note that there is only one pod running since the manifest specified replicas=1.
```shell
❯ kubectl get pods -l tier=frontend
NAME                        READY   STATUS    RESTARTS   AGE
frontend-54bb48c447-jp87x   1/1     Running   0          34s
```

In a new shell window, watch for pods.  In another window deploy, the hpa. And then notice the new pods being created becuase the HPA specifies a minimum replica count of 2.
```shell

shell 1❯ kubectl get pods -w
NAME                        READY   STATUS    RESTARTS   AGE
frontend-54bb48c447-jp87x   1/1     Running   0          4m25s

shell 2❯ kubectl apply -f 06-frontend-hpa-v1-api.yaml 
horizontalpodautoscaler.autoscaling/frontend-v1 created

shell 1❯ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
frontend-54bb48c447-jp87x   1/1     Running   0          8m6s
frontend-54bb48c447-kbmmw   1/1     Running   0          87s
```

Now let's put the pods under some load and see what happens.

First take note of the HPA.  There shouldn't be any load.
```shell
❯ kubectl get hpa frontend-v1
NAME          REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
frontend-v1   Deployment/frontend   0%/50%    2         5         2          6m50s
```

Startup the load generator
```shell
> kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh

while true; do wget -q -O- http://frontend.default.svc.cluster.local; done
```

After about 30 seconds or so, check the HPA again and if you haven't closed out of a second terminal you can watch the pods spin up.
```shell
shell 1❯ kubectl get hpa frontend-v1
NAME          REFERENCE             TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
frontend-v1   Deployment/frontend   100%/50%   2         3         3          10m

shell 2❯ k get pods -w
NAME                        READY   STATUS    RESTARTS   AGE
frontend-54bb48c447-8wjnx   1/1     Running   0          71s
frontend-54bb48c447-gmccd   1/1     Running   0          2m58s
frontend-54bb48c447-jp87x   1/1     Running   0          19m
frontend-54bb48c447-kbmmw   1/1     Running   0          12m
load-generator              1/1     Running   0          4m22s
```
Kubernetes will do it's best to spin up enough pods withing the constraints of the HPA definition to bring the target CPU% usage under the target amount.  This make take some time and at what point things settle down will depend on the cluster resources i.e. how many nodes and how many CPUs, etc.

```shell
❯ kubectl get hpa frontend-v1
NAME          REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
frontend-v1   Deployment/frontend   47%/50%   2         5         4          13m
```

Stop the load generator.  First Ctrl-C, and then exit.
```shell
...
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
^C
# exit
Session ended, resume using 'kubectl attach load-generator -c load-generator -i -t' command when the pod is running
pod "load-generator" deleted
```

> It'll take five minutes for Kubernetes to settle things back down to just two replicas.  This is because of the cooldown period.

### Clean Up

Delete the HPA but leave the deployment as they'll be reused in the next lab.
```shell
❯ kubectl delete hpa frontend-v1
horizontalpodautoscaler.autoscaling "frontend-v1" deleted
```

Next: [Horizontal Scaling - Memory Lab](07-horizontal-scaling-mem.md)
