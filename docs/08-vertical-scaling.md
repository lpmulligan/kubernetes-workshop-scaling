# Vertical Pod Autoscaling (VPA)

Vertical Pod Autoscaler (VPA) frees the users from necessity of setting up-to-date resource limits and requests for the containers in their pods. When configured, it will set the requests automatically based on usage and thus allow proper scheduling onto nodes so that appropriate resource amount is available for each pod. It will also maintain ratios between limits and requests that were specified in initial containers configuration.

It can both down-scale pods that are over-requesting resources, and also up-scale pods that are under-requesting resources based on their usage over time.

Autoscaling is configured with a Custom Resource Definition object called VerticalPodAutoscaler. It allows to specify which pods should be vertically autoscaled as well as if/how the resource recommendations are applied.

> Vertical Pod Autoscaling is a recent feature and is not yet available in AKS.  However, it does work and may be able to assist with sizing pod requests and limits.

## Installation

Follow, the install documents [here](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler).


## Examples

Follow the examples [here](https://cloud.google.com/kubernetes-engine/docs/how-to/vertical-pod-autoscaling).

## Links

* [Replex Blog Post](https://www.replex.io/blog/kubernetes-in-production-best-practices-for-cluster-autoscaler-hpa-and-vpa)
* [LevelUp Blog Post](https://levelup.gitconnected.com/kubernetes-autoscaling-101-cluster-autoscaler-horizontal-pod-autoscaler-and-vertical-pod-2a441d9ad231)
* [Vertical Pod Autoscaling](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
* [GKE Docs for VPA](https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler)