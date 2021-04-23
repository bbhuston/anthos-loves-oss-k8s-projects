# Using OPA Constraints


##### 1) Create example namespace
```
kubectl create ns opa-constraint-example
```

##### 2) Install OPA constraint templates
```
kubectl apply -f opa-constraints.yaml
```

##### 3) Install example apps that violate these constraints
```
kubectl apply -f bad-example-no-resource-limits.yaml -f bad-example-no-probes.yaml
```

##### 4) Confirm that the OPA admission controller blocked this app
```
kubectl -n opa-constraint-example get deployments
```

##### 5) Install good example app that does not violate constraints
```
kubectl apply -f good-example-with-limits-and-probes.yaml
```

##### 6) Confirm that the OPA admission controller did not block this app
```
kubectl -n opa-constraint-example get deployments
```
