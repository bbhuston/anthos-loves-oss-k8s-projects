# Kubeform Installation Steps

##### 1) Install kubeform operator and CRDs

```
kubectl create ns kubeform
kubectl apply -f kubeform-v0.3.0.yaml -f aws-crds-0.3.0.yaml
```

##### 2) Install kubeform license and AWS account credential secrets


```
# Generate secrets
cat << EOF > kubeform-secrets.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: aws
  namespace: kubeform
type: kubeform.com/aws
data:
  # Note: Fill in these values with AWS access key information
  region: <base64 encoded value of `us-east-1`>
  access_key: <base64 encoded AWS access key with EC2 IAM permissions>
  secret_key: <base64 encoded AWS secret key with EC2 IAM permissions>
EOF

---
apiVersion: v1
kind: Secret
metadata:
  name: kubeform-license
  namespace: kubeform
  labels:
    helm.sh/chart: kubeform-v0.3.0
    app.kubernetes.io/name: kubeform
    app.kubernetes.io/instance: kubeform
    app.kubernetes.io/version: "v0.3.0"
    app.kubernetes.io/managed-by: Helm
type: Opaque
data:
  # NOTE: Generate a seperate community license per cluster as per
  # these instructions: https://kubeform.cloud/docs/v2020.10.30/setup/install/community/
  key.txt: <Base64-encoded version of license key>
EOF

# Apply secrets
kubectl -n kubeform apply -f kubeform-secrets.yaml

```
##### 3) Confirm that kubeform is running

```
kubectl -n kubeform get pods
```

##### 4) Confirm that example AWS EC2 can be created via kubeform

```
kubectl apply -f aws-ec2-example.yaml

kubectl -n kubeform get Instance
```
