# Kubevirt Installation Steps

##### 1) Install kubevirt operator and CRDs

```
kubectl apply -f kubevirt-162.yaml
kubectl apply -f cdi-162.yaml
```

##### 2) Install virtctl plugin for kubectl
```
# install krew package manager
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"${OS}_${ARCH}" &&
  "$KREW" install krew
)
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

# install virctl plugin for kubectl
kubectl krew install virt
kubectl virt
```

##### 3) Create example virtualmachine
```
# NOTE: Kubevirt on ABM may require a different custom VM image than used in this example
kubectl -n kubevirt apply -f example-vm.yaml
```

##### 4) Start, stop, and SSH into VM
```
# Check VMs
kubectl get vms

# Start VMs
kubectl virt start testvm

# Check VM instances
kubectl get vmi

# Connect to VM instance
kubectl virt console testvm

# Stop VM
kubectl virt stop testvm
```