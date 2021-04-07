# Knative Installation Steps

IMPORTANT: This steps below assume Istio is already installed on your Kubernetes cluster.


##### 1) Install the knative operators and CRDs

```
kubectl apply -f knative-operator-v0.19.4.yaml
kubectl apply -f knative.yaml
```

##### 2) Add Istio resources for knative to use
```
# This fixes the "IngressNotConfigured" error that knative will display for services 
kubectl apply -f net-istio-v0.20.0.yaml
```

##### 3) Install an example knative service
```
kubectl create ns cloudrun-example
kubectl -n cloudrun-example apply -f cloudrun-example.yaml
kubectl get ksvc -A
```

##### 4) Install the hey load testing tool
```
curl -o hey  https://hey-release.s3.us-east-2.amazonaws.com/hey_linux_amd64
chmod +x hey
```

##### 5) Run a load test against the knative service
```
# Run `kubectl -n istio-system get svc -l app=istio-ingressgateway`
# Retreve EXTERNAL-IP and set this value for ISTIO_INGRESSGATEWAY_IP
ISTIO_INGRESSGATEWAY_IP=<set value from EXTERNAL-IP>
LOAD_TEST_HOSTNAME=cloudrun-hello-world.cloudrun-example.example.com

# Run the load test. The command below creates 70 concurrent processes which all send requests for 60 seconds
./hey -z 60s -c 70 -host "${LOAD_TEST_HOSTNAME}" "http://${ISTIO_INGRESSGATEWAY_IP}?sleep=100&prime=10000&bloat=5"
```