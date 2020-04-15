# Kubernetes Resource Limits and Quality of Service (QoS)


## Resource Limits

All information about resource limits can be found on the Kubernetes documentation page for[Manage Compute Resources](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/).

## QoS

All information about QoS can be found on the Kubernetes documentation page for [Configure Quality of Service for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/).  It important to read and undertands what is presented there.

### Additional QoS information

However a bit more explanation about the importance of QoS is in order, since it relates to scaling.

Most of the Pod definitions examples ignore the requests and limits parameters. You are not strictly required to include them when designing your cluster. Adding or ignoring requests and limits affects the Quality of Service (QoS) that the Pod receives as follows:

* **Lowest priority pods:** when you do not specify requests and limits, the Kubelet will deal with your Pod in a best-effort manner. The Pod, in this case, has the lowest priority. If the node runs out of non-shareable resources, best-effort Pods are the first to get killed.

* **Medium priority pods:** if you define both parameters and set the requests to be less than the limit, then Kubernetes manages your Pod in the Burstable manner. When the node runs out of non-shareable resources, the Burstable Pods will get killed only when there are not more best-effort Pods running.

* **Highest priority pods:** your Pod will be deemed as of the most top priority when you set the requests and the limits to equal values. It’s as if you’re saying, “I need this Pod to consume no less and no more than x memory and y CPU.” In this case, and in the event of the node running out of shareable resources, Kubernetes does not terminate those Pods until the best-effort, and the burstable Pods are terminated. Those are the highest priority Pods.

We can summarize how the Kubelet deals with Pod priority as follows:

![QoS Table](images/QoS-table.png)

### Priority Considerations

#### Pod Priority

Sometimes you may need to have more fine-grained control over which of your Pods get evicted first in the event of resource starvation. You can guarantee that a given Pod get evicted last if you set the request and limit to equal values. However, consider a scenario when you have two Pods, one hosting your core application and another hosting its database. You need those Pods to have the highest priority among other Pods that coexist with them. But you have an additional requirement: you want the application Pods to get evicted before the database ones do. Fortunately, Kubernetes has a feature that addresses this need: Pod Priority and preemption. Pod Priority and preemption is stable as of Kubernetes 1.14 or higher. 

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
 name: high-priority
value: 1000
---
apiVersion: v1
kind: Pod
metadata:
 name: mypod
spec:
 containers:
 - image: redis
   name: mycontainer
 priorityClassName: high-priority
```

### TL;DR 
* Scaling is an important operational practice that used to be done manually for a long period of time. With the introduction of HorizontalPodAutoscaler (HPA), Kubernetes can take intelligent scaling decisions automatically.

* There are two types of scaling: horizontal scaling, which refers to increasing the number of Pods serving the application, and Vertical scaling which refers to expanding the resources of the Pods.

* HPA can be easily applied to resources that can be throttled like CPU. However, when you need to use non-throttable resources like memory, the process becomes more complex as the application needs to be aware that more Pods were spawned and start to release memory. Otherwise, the HPA continues spawning Pods until the upper limit is reached.

* Vertical Pod Autoscaling is still experimental at the time of this writing. When implemented, it can modify the requests and limits value of Pods based on their consumption levels. However, this needs much consideration than HPA because changing the requests and limits value requires deleting and recreating Pods. Additionally, the application must be carefully designed so that it utilizes the new resources. For example, JVM defines the maximum and minimum memory that it would consume through command line arguments.


### Links

* [Kubernetes QoS](https://medium.com/better-programming/the-kubernetes-quality-of-service-conundrum-eebbbb5f89cf)

Next: [QoS Lab](04-qos-lab.md)