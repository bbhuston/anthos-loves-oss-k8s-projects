# Using OPA Constraints

NOTE:  One needs to have Policy Controller (aka OPA) installed on your cluster with the ["template library" enabled](https://cloud.google.com/anthos-config-management/docs/how-to/installing-policy-controller#installing) in order for this example to work properly.

##### 1) Create example namespaces
```
kubectl create ns opa-constraint-example
kubectl create ns opa-resourcequota-example
```

##### 2) Enable OPA constraints
```
kubectl apply -f extra-opa-constraint-templates.yaml -f opa-constraints.yaml -f opa-custom-constraint-config.yaml
```

##### 3) Install example apps that violate these constraints
```
kubectl apply -f bad-example-no-resource-limits.yaml -f bad-example-no-probes.yaml
```

##### 4) Confirm that the OPA admission controller blocked this app

The output returned from the command below will show that bad deployments are  blocked from being scheduled because they violate the policy constraints.

```
kubectl -n opa-constraint-example get deployments
kubectl -n opa-constraint-example get event
```

##### 5) Install good example app that does not violate constraints
```
kubectl apply -f good-example-with-limits-and-probes.yaml
```

##### 6) Confirm that the OPA admission controller did not block this app

The output returned from the command below will show that the good deployment was successfully scheduled.

```
kubectl -n opa-constraint-example get deployments
kubectl -n opa-constraint-example get event
```

##### 7) Install example apps that violate the namespace resource quota constraint
```
kubectl apply -f bad-example-no-namespace-resource-quota.yaml
```

##### 8) Confirm that the OPA admission controller blocked this app

The output returned from the command below will show that bad deployment is blocked from being scheduled because it violates the policy constraint.

```
kubectl -n opa-resourcequota-example get deployments
kubectl -n opa-resourcequota-example get event
```

##### 9) Add a ResourceQuota to the example namespace
```
kubectl apply -f resource-quota.yaml
```

##### 10) Confirm that the OPA admission controller stopped blocking this app
```
kubectl -n opa-resourcequota-example get deployments
kubectl -n opa-resourcequota-example get event
```