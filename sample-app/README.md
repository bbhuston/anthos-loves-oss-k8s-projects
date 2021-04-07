# Sample App Installation Steps


##### 1) Create namespace
```
kubectl create ns demo
```

##### 2) Install sample app
```
kubectl -n demo apply -f manifests.yaml -f istio-manifests.yaml
```

##### 3) Confirm that app is running
```
kubectl -n demo get pods
```

##### 3) Optional: Connect to frontend service via a local proxy 
```
# If a web browser is enabled on your workstation view the app with the following commands
kubectl -n demo port-forward svc/frontend 8085:80 &

# Navigate to "http://localhost:8085" in your browser

# Terminate the proxy when you are finished
killall kubectl
```