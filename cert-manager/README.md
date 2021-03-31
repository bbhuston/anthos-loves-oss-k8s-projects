# Cert-Manager Installation Steps

##### 1) Install cert-manager operator and CRDs
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.2.0/cert-manager.yaml
```

##### 2) Confirm cert-manager is running

```
kubectl get pods --namespace cert-manager
```

##### 3) Confirm that a cert-manager example is working

```
kubectl apply -f cert-manager-example.yaml
```

```
kubectl describe certificate -n cert-manager-test
```