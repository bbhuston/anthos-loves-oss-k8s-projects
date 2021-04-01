# ExternalSecrets Installation Steps

##### 1) Configure ExternalSecrets installation prerequsites

```
# Create namespace
kubectl create ns external-secrets

```

```
# Generate AWS credentials secret
cat << EOF > aws-secret.yaml
---
apiVersion: v1
data:
  # Need to generate AWS access key with AWS Secret Manager IAM permissions
  # Base64 encode AWS access key id
  id: cmVwbGFjZW1lCg==
  # Base64 encode AWS secret key
  key: cmVwbGFjZW1lCg==
kind: Secret
metadata:
  name: aws-credentials
  namespace: external-secrets
type: Opaque
EOF

# Generate AWS credentials secret
kubectl -n external-secrets apply -f aws-secret.yaml

# Optional: Create GCP service account and key with "Secret Manager Admin" in GCP Console
export GCP_SA_KEYFILE_CONTENT=<Paste SA key content here>

# Optional: Generate GCP secret
cat << EOF > gcp-secret.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: gcp-creds
type: Opaque
stringData:
  gcp-creds.json: |-
    ${GCP_SA_KEYFILE_CONTENT}
EOF

# Optional: Generate GCP credentials secret
kubectl -n external-secrets apply -f gcp-secret.yaml

```

##### 2) Install ExternalSecrets operator and CRDs
```
kubectl -n external-secrets apply -f external-secrets-v6.1.0.yaml
```

##### 3) Confirm ExternalSecrets is running
```
kubectl -n external-secrets get pods
```

##### 4) Confirm that example AWS ExternalSecret works
```
# Create example secret in AWS Secret Manager
aws secretsmanager create-secret --name hello-service/password --secret-string "1234"

cat << EOF > aws-example-secret.yaml
---
apiVersion: 'kubernetes-client.io/v1'
kind: ExternalSecret
metadata:
  name: hello-service
  namespace: external-secrets
spec:
  backendType: secretsManager
  # optional: specify role to assume when retrieving the data
#  roleArn: arn:aws:iam::123456789012:role/test-role
  data:
    - key: hello-service/password
      name: password
  # optional: specify a template with any additional markup you would like added to the downstream Secret resource.
  # This template will be deep merged without mutating any existing fields. For example: you cannot override metadata.name.
  template:
    metadata:
      annotations:
        example-annotation: value1
      labels:
        example-label: value2
EOF

# Create ExternalSecret
kubectl -n external-secrets apply -f aws-example-secret.yaml

# Check ExternalSecret
kubectl -n external-secrets get ExternalSecret hello-service -o yaml
```


##### 5) Optional: Confirm that example GCP ExternalSecret works

```
# Create example secret in GCP Secret Manager
echo -n '{"secret-manager-key-name": "my-secret-value"}' |     gcloud secrets create external-secrets-example-001     --replication-policy="automatic"     --data-file=-

cat << EOF > gcp-example-secret.yaml
---
apiVersion: kubernetes-client.io/v1
kind: ExternalSecret
metadata:
  name: gcp-secrets-manager-example    # name of the k8s external secret and the k8s secret
  namespace: external-secrets
spec:
  backendType: gcpSecretsManager
  projectId: ${PROJECT_ID}
  # You can create this secret in GCP Secret Manager using the GCP console or gclou
  data:
    - key: external-secrets-example-001     # name of the GCP secret
      name: k8s-key-name    # key name in the k8s secret
      version: latest    # version of the GCP secret
      property: secret-manager-key-name      # name of the field in the GCP secret
EOF

# Create ExternalSecret
kubectl -n external-secrets apply -f gcp-example-secret.yaml

# Check ExternalSecret
kubectl -n external-secrets get ExternalSecret gcp-secrets-manager-example -o yaml
```