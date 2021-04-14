# Velero Installation Steps

NOTE:  The steps below assume that one has already completed the instructions described in `velero/README.md`, such as downloading the velero cli and using it to install the velero CRDs on your target cluster. 

##### 1) Set GCP project
```
PROJECT_ID=<Enter GCP project ID here>

gcloud auth login
gcloud config set project $PROJECT_ID
gcloud config list project
```

##### 2) Create a Velero backup

NOTE: To back up all resources on the cluster omit the `--include-namespaces` flag.

```
# Backup the demo namespace
velero backup create example-backup-001-vz-testing-demo --include-namespaces demo

# Check progress of the backup
velero backup describe example-backup-001-vz-testing-demo
```

##### 3) Delete the sample application

Note: If the sample application is not already installed in your environment, please follow the steps described in `sample-app/README.md`
```
# List the sample app deployments
kubectl -n demo get deployment

# Delete the deployments
kubectl -n demo delete deployment --all

# Confirm deletion
kubectl -n demo get deployment
```

##### 4) Restore the deleted application using Velero

```
# Start a restore job
velero restore create example-restore-001-vz-testing-demo --from-backup example-backup-001-vz-testing-demo

# Monitor the restore job
velero restore describe example-restore-001-vz-testing-demo
```

##### 5) Confirm that sample application is running once again on the cluster

```
kubectl -n demo get deployment
```

##### 6) Optional: Further reading

- To see how Velero can be used to specifically backup just etcd across different source and destination clusters, please [see this tutorial](https://docs.bitnami.com/tutorials/backup-restore-data-etcd-kubernetes/).
- To see how the `etcdctl` utility can directly be used to backup and restore etcd on Anthos on VMware clusters, please see [this guide](https://cloud.google.com/anthos/clusters/docs/on-prem/1.6/how-to/backing-up). 