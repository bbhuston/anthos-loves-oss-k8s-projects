# Velero Installation Steps


##### 1) Set GCP project
```
PROJECT_ID=<Enter GCP project ID here>

gcloud auth login
gcloud config set project $PROJECT_ID
gcloud config list project
```

##### 2) Install velero cli
```
wget -O velero-v1.5.3-linux-amd64.tar.gz https://github.com/vmware-tanzu/velero/releases/download/v1.5.3/velero-v1.5.3-linux-amd64.tar.gz velero-v1.5.3-linux-amd64.tar.gz
tar -xvf velero-v1.5.3-linux-amd64.tar.gz
chmod +x velero-v1.5.3-linux-amd64/velero
mv velero-v1.5.3-linux-amd64/velero /usr/local/bin
velero version
```

##### 3) Create backup bucket
```
BUCKET=<Enter globally unique bucket name for Velero backups>
gsutil mb gs://$BUCKET/
```

##### 4) Setup GCP service account  for Velero

*OPTION A:* Use preexisting GCP service account for Velero

```
export VELERO_SA_KEY_PATH=<Enter filepath of your GCP service account json key>
```

*OPTION B:* Create GCP service for Velero and attach IAM permissions

```
gcloud iam service-accounts create velero --display-name "Velero service account"
SERVICE_ACCOUNT_EMAIL=velero@${PROJECT_ID}.iam.gserviceaccount.com

ROLE_PERMISSIONS=(
    compute.disks.get
    compute.disks.create
    compute.disks.createSnapshot
    compute.snapshots.get
    compute.snapshots.create
    compute.snapshots.useReadOnly
    compute.snapshots.delete
    compute.zones.get
)

gcloud iam roles create velero.server \
    --project $PROJECT_ID \
    --title "Velero Server" \
    --permissions "$(IFS=","; echo "${ROLE_PERMISSIONS[*]}")"

gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:${SERVICE_ACCOUNT_EMAIL} \
    --role projects/$PROJECT_ID/roles/velero.server

gsutil iam ch serviceAccount:${SERVICE_ACCOUNT_EMAIL}:objectAdmin gs://${BUCKET}
gcloud iam service-accounts keys create credentials-velero --iam-account $SERVICE_ACCOUNT_EMAIL

export VELERO_SA_KEY_PATH=./credentials-velero
```

##### 5) Install Velero operator and CRDs
```
velero install \
    --provider gcp \
    --plugins velero/velero-plugin-for-gcp:v1.1.0 \
    --bucket $BUCKET \
    --secret-file ${VELERO_SA_KEY_PATH}
```

##### 6) Create Velero backup
```
velero backup create example-backup-001
velero backup describe example-backup-001
```
