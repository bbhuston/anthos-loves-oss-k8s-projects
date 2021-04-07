# VPA Installation Steps


##### 1) Connect to your user cluster
```
kubectl config use-context <your cluster context>
```

##### 2) Install VPA
```
git clone https://github.com/kubernetes/autoscaler.git
 ./autoscaler/vertical-pod-autoscaler/hack/vpa-up.sh
 ./autoscaler/vertical-pod-autoscaler/pkg/admission-controller/gencerts.sh

```
##### 3) Create an example VPA resource

First, follow the README in the `sample-app` directory to install an application that the VPA resource can reference.

Next, install the example VPA resource
```
kubectl -n demo apply -f vpa-example.yaml
kubectl -n demo get verticalpodautoscalers.autoscaling.k8s.io
kubectl -n demo describe verticalpodautoscalers.autoscaling.k8s.io
```

The VPA example autoscales the `loadgenerator` deployment in the sample app. Edit the `USERS` environmental variable that is set in the `loadgenerator` deployment to increase the workload for this service and cause VPA to make recommendations. An example recommendation will eventually appear in the `kubectl describe` output, as per the output below.

```
  Recommendation:
    Container Recommendations:
      Container Name:  main
      Lower Bound:
        Cpu:     9m
        Memory:  34603008
      Target:
        Cpu:     15m
        Memory:  40894464
      Uncapped Target:
        Cpu:     15m
        Memory:  40894464
      Upper Bound:
        Cpu:     1120m
        Memory:  3272605696
```
 

##### 5) Optional: Monitor the VPA controllers
```
kubectl -n kube-system get pod | grep vpa
kubectl -n kube-system logs -l app=vpa-admission-controller
kubectl -n kube-system logs -l app=vpa-recommender
kubectl -n kube-system logs -l app=vpa-updater
```

