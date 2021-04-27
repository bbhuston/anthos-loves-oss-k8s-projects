##### Example manifest for usage metering agent

```
# NOTE: The values listed below are just a notional example.
# This resource will not run "as is" if installed on a cluster.
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: gke-usage-metering
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gke-usage-metering
  template:
    metadata:
      labels:
        app: gke-usage-metering
      name: gke-usage-metering
      namespace: kube-system
    spec:
      containers:
      - args:
        - --cluster-name=demo
        - --bigquery-project-id=ziyuanchen-gke-dev
        - --bigquery-dataset-id=syllogi_dev
        - --bigquery-table-id=gke_cluster_resource_usage
        - --enabled-usage-recorders=pod,PodMetric
        - --enabled-exporters=bigquery
        - --metrics-port=9099
        - --persistent-dir=/tmp/gke-usage-metering
        - --cluster-environment=on-prem
        - --bigquery-consumption-table-id=gke_cluster_resource_consumption
        command:
        - /gke-resource-tracker
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value: /etc/bq-creds/ziyuanchen-gke-dev-bq-access.json
        image: gcr.io/gke-on-prem-staging/gke-resource-tracker:v0.7.0
        imagePullPolicy: Always
        name: gke-usage-metering
        volumeMounts:
        - mountPath: /tmp/gke-usage-metering
          name: gke-usage-metering-local-storage
        - mountPath: /etc/bq-creds
          name: bq-creds
          readOnly: true
      imagePullSecrets:
      - name: private-registry-creds
      restartPolicy: Always
      schedulerName: default-scheduler
      serviceAccount: gke-usage-metering-sa
      serviceAccountName: gke-usage-metering-sa
      volumes:
      - hostPath:
          path: /tmp/gke-usage-metering
          type: DirectoryOrCreate
        name: gke-usage-metering-local-storage
      - name: bq-creds
        secret:
          secretName: usage-metering-bigquery-service-account-key
```