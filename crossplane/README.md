# Crossplane Installation Steps

##### 1) Install crossplane operator and CRDs

```
kubectl create ns crossplane-system
kubectl -n crossplane-system apply -f meta-crds/
kubectl -n crossplane-system apply -f crossplane-v0.14.0.yaml -f meta-crds/providerconfigs-crds

```

##### 2) Install crossplane AWS secret

```
# Generate secret
cat << EOF > aws-secret.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-creds
  namespace: crossplane-system
type: Opaque
data:
  # Base64 encoded of AWS account credentials from "aws configure" CLI command
  key: cmVwbGFjZW1lCg==
EOF

# Apply secret
kubectl -n crossplane-system apply -f aws-secret.yaml
```

##### 3) Install crossplane AWS ProviderConfig 
```
kubectl -n crossplane-system apply -f provider-aws.yaml
```

##### 4) Confirm that crossplane is running

```
kubectl -n crossplane-system get pods
```

##### 5) Optional: Create example EKS cluster

```
# NOTE: This requires the AWS IAM permissions listed in the example manifest!
# It also takes a few mintues for the Crossplane AWS CRDs to finish installing.
# Please rerun the `kubectl apply` listed below if you encounter a missing CRD 
# error message the first time you run the command.

kubectl -n crossplane-system apply -f aws-eks-example.yaml

# Confirm EKS cluster is healthy
kubectl -n crossplane-system describe Cluster eks-cluster-001
```
