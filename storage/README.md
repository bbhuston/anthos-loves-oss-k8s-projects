# Persistent Volumes

##### 1) Display storage class names
```
kubectl get sc
```

##### 2) Create persistent volume claim
```
# Set storage class name
export STORAGE_CLASS=<Enter storage class name here (i.e., "premium-rwo")>

# Generate manifests
cat << EOF > pvc.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: "${STORAGE_CLASS}"
EOF

# Apply manifests
kubectl apply -f pvc.yaml
```

##### 3) Confirm volumes were created
```
kubectl get pvc,pv
```
