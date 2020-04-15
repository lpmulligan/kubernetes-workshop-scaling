# Kubernetes Resource Limits and Quality of Service (QOS) Lab

## QOS

Perform the following steps to see Kubernetes QOS in action. 

### Best Effort
```shell
# Deploy the pod
❯ kubectl apply -f 01-besteffort-qos-nginx-pod.yaml 
pod/nginx-besteffort created

# Check that the pod is running
❯ kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
nginx-besteffort   1/1     Running   0          5s

# Check the resources
> kubectl get pod nginx-besteffort --output=yaml
spec:
  containers:
  ...
    resources: {}
...
qosClass: BestEffort

!! Notice 'resources{}' is empty

# Check the QOS Class Only
❯ kubectl get pod nginx-besteffort --output=jsonpath='{.status.qosClass}'
BestEffort
```

### Burstable
```shell
# Deploy the pod
❯ kubectl apply -f 01-burstable-qos-nginx-pod.yaml
pod/nginx-burstable created

# Check that the pod is running
❯ kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
nginx-besteffort   1/1     Running   0          3m49s
nginx-burstable    1/1     Running   0          5s

# Check the resources
> kubectl get pod nginx-burstable --output=yaml
...
spec:
  containers:
  ...
    resources:
      limits:
        cpu: 200m
        memory: 100Mi
      requests:
        cpu: 100m
        memory: 50Mi
...
qosClass: Burstable

# Check the QOS Class Only
❯ kubectl get pod nginx-burstable --output=jsonpath='{.status.qosClass}' 
Burstable 
```

### Guaranteed

```shell
# Deploy the pod
❯ kubectl apply -f 01-guaranteed-qos-nginx-pod.yaml 
pod/nginx-quaranteed created

# Check that the pod is running
❯ kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
nginx-besteffort   1/1     Running   0          30m
nginx-burstable    1/1     Running   0          25m
nginx-guaranteed   1/1     Running   0          16s

# Check the resources
> kubectl get pod nginx-guaranteed --output=yaml
...
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx-guaranteed
    resources:
      limits:
        cpu: 100m
        memory: 100Mi
      requests:
        cpu: 100m
        memory: 100Mi
...
qosClass: Guaranteed

# Check the QOS Class
❯ kubectl get pod nginx-guaranteed --output=jsonpath='{.status.qosClass}' 
Guaranteed 
```

> Expanding upon searching for a pod's QOS.  You can also search QOS for all pods in in cluster.
```shell
❯ kubectl get pods --all-namespaces -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,QOS-CLASS:.status.qosClass

NAME                                        NAMESPACE     QOS-CLASS
nginx-besteffort                            default       BestEffort
nginx-burstable                             default       Burstable
nginx-guaranteed                            default       Guaranteed
aci-connector-linux-fbf85d75b-rsd5p         kube-system   BestEffort
azure-cni-networkmonitor-5zljw              kube-system   BestEffort
azure-ip-masq-agent-xn6qg                   kube-system   Burstable
azure-policy-8597b94c6c-g5g9p               kube-system   Burstable
coredns-5448687fdb-7zhb8                    kube-system   Guaranteed
...
```

### Clean Up

```shell
> kubectl delete pods nginx-besteffort nginx-burstable nginx-guaranteed
```

Next: [Horizontal Scaling](05-horizontal-scaling.md)
