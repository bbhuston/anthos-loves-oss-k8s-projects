# VPA Installation Steps


##### 1) Connect to your user cluster
```
kubectl config use-context <your cluster context>
```

##### 2) Install VPA
```
git clone https://github.com/kubernetes/autoscaler.git
 ./autoscaler/vertical-pod-autoscaler/hack/vpa-up.sh
```

##### 3) Create example VPA object

```
# NOTE: The VPA example is not referencing a real k8s deployment here, so VPA isn't really autoscaling anything
kubectl apply -f vpa-example.yaml
kubectl get verticalpodautoscalers.autoscaling.k8s.io -A
```