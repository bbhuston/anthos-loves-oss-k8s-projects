# Configure Usage Metering with BigQuery


##### 1) Prepare GCP IAM prerequisites

NOTE: Skip this step if you already have a GCP service account, and JSON API key, with the neccessary BigQuery permissions ready

```
export PROJECT="Your GCP Project ID"

# Create a GCP service account
gcloud iam service-accounts create bigquery-data-editor --project ${PROJECT}

# Bind IAM role to service account
gcloud projects add-iam-policy-binding ${PROJECT} --member="serviceAccount:bigquery-data-editor@${PROJECT}.iam.gserviceaccount.com" --role="roles/bigquery.dataEditor"

# Generate key for service account
gcloud iam service-accounts keys create bigquery-key.json --iam-account  bigquery-data-editor@${PROJECT}.iam.gserviceaccount.com
```

##### 2) Prepare GCP BiqQuery prerequisites

NOTE:  You need to create a dataset in BigQuery that can be used to store metering data.

```
# Set GCP environmental variables that will be used by the agent
export BIGQUERY_PROJECT_ID=""
export BIGQUERY_DATASET_ID="" # something like "gke_usage_metering"
export BIGQUERY_TABLE_NAME=""  # something like "gke_onprem_prices"

# Create BQ dataset
bq --project_id=${BIGQUERY_PROJECT_ID} mk -d --location=US --description "GKE Usage Metering Dataset" ${BIGQUERY_DATASET_ID}
```

##### 3) Install the usage metering agent's RBAC resources
```
kubectl apply -f gke-usage-metering-sa.yaml -f gke-usage-metering-clusterrole.yaml -f gke-usage-metering-clusterrolebinding.yaml
```

##### 4) Install the usage metering agent
```
# Set additional GCP environmental variables that will be used by the agent
export CLUSTER_NAME=""
export PROXY_URL=""
export GCP_SA_KEY_NAME=""  # something like "bigquery-key.json"
export PATH_TO_GCP_SA_KEY=""  # something like "./bigquery-key.json" if this key is in the current directory

# Create a Kubernetes secret to be used by the agent
kubectl -n kube-system create secret generic usage-metering-bigquery-service-account-key --from-file=${PATH_TO_GCP_SA_KEY}

# Install the agent

# Update placeholder values for the agent deployment
sed -i -e 's|${CLUSTER_NAME}|'$CLUSTER_NAME'|g' gke-usage-metering.yaml
sed -i -e 's|${BIGQUERY_PROJECT_ID}|'$BIGQUERY_PROJECT_ID'|g' gke-usage-metering.yaml
sed -i -e 's|${BIGQUERY_DATASET_ID}|'$BIGQUERY_DATASET_ID'|g' gke-usage-metering.yaml
sed -i -e 's|${PROXY_URL}|'$PROXY_URL'|g' gke-usage-metering.yaml
sed -i -e 's|${GCP_SA_KEY_NAME}|'$GCP_SA_KEY_NAME'|g' gke-usage-metering.yaml

# install the agent
kubectl apply -f gke-usage-metering.yaml

# Confirm that the agent is running
kubectl -n kube-system get pod -l app=gke-usage-metering
```

##### 5) Add custom pricing data to BigQuery
```
# Update placeholder values
sed -i -e 's|${BIGQUERY_PROJECT_ID}|'$BIGQUERY_PROJECT_ID'|g' $(pwd)/queries/create-price-table
sed -i -e 's|${BIGQUERY_DATASET_ID}|'$BIGQUERY_DATASET_ID'|g' $(pwd)/queries/create-price-table
sed -i -e 's|${BIGQUERY_TABLE_NAME}|'$BIGQUERY_TABLE_NAME'|g' $(pwd)/queries/create-price-table
sed -i -e 's|${BIGQUERY_PROJECT_ID}|'$BIGQUERY_PROJECT_ID'|g' $(pwd)/queries/join-bq-tables
sed -i -e 's|${BIGQUERY_DATASET_ID}|'$BIGQUERY_DATASET_ID'|g' $(pwd)/queries/join-bq-tables
sed -i -e 's|${BIGQUERY_TABLE_NAME}|'$BIGQUERY_TABLE_NAME'|g' $(pwd)/queries/join-bq-tables

# Create BQ schemas
bq --project_id=${BIGQUERY_PROJECT_ID} query --use_legacy_sql=false --flagfile=$(pwd)/queries/create-price-table

# Join your resource prices with monitoring data
bq query --project_id=${BIGQUERY_PROJECT_ID} --use_legacy_sql=false --flagfile=$(pwd)/queries/join-bq-tables
```

##### 6) Connect the DataStudio Anthos usage metering dashboard to BigQuery

Finally, go to [this URL](https://datastudio.google.com/c/u/0/reporting/d7374e12-54aa-438c-a9ab-dabcb90e83be) and copy a DataStudio dashboard for usage metering.