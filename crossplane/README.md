# Crossplane Installation Steps

##### 1) Install crossplane operator and CRDs

```
kubectl create ns crossplane-system
kubectl -n crossplane-system apply -f meta-crds/
kubectl -n crossplane-system apply -f crossplane-v0.14.0.yaml
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
  # Need base64 encoded secret
  key: <base64 encoded of AWS account credentials from "aws configure" CLI command>
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
kubectl -n crossplane-system apply -f aws-eks-example.yaml

# Confirm EKS cluster is healthy
kubectl -n crossplane-system desribe Cluster eks-cluster-001
```
