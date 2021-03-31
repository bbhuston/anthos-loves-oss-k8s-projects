# External-DNS Installation Steps

> v0.7.6

### Apply CRD for external-dns

- Base CRD has been modified:
    - Added a namespace called external-dns
    - Set service account external-dns to ns external-dns
    - Set deployemnt external-dns to ns external-dns
- Currently set to aws provider...
  - Configure the --domain-filter
  - Configure the --txt-owner-id=

```
# External DNS with RBAC for ISTIO-GTW
kubectl apply -f external-dns-istio-gw-crd.yaml
```