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
bq mk -d --project_id=${BIGQUERY_PROJECT_ID} --location=US --description "GKE Usage Metering Dataset" ${BIGQUERY_DATASET_ID}
```

##### 3) Install the usage metering agent's RBAC resources
```
kubectl apply -f usage-metering/gke-usage-metering-sa.yaml -f usage-metering/gke-usage-metering-clusterrole.yaml -f usage-metering/gke-usage-metering-clusterrolebinding.yaml
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
kubectl apply -f usage-metering/gke-usage-metering.yaml

# Confirm that the agent is running
kubectl -n kube-system get pod -l app=gke-usage-monitoring
```

##### 5) Add custom pricing data to BigQuery
```
# Create BQ schemas

bq query --use_legacy_sql=false \
    'CREATE TABLE `${BIGQUERY_PROJECT_ID}.${BIGQUERY_DATASET_ID}.${BIGQUERY_TABLE_NAME}`
    (
      sku_id STRING,
      resource_name STRING,
      rate FLOAT64,
      unit STRING
    );
    
    INSERT INTO `${BIGQUERY_PROJECT_ID}.${BIGQUERY_DATASET_ID}.${BIGQUERY_TABLE_NAME}`
    VALUES (NULL, 'cpu', 0.00001, 'core-second');

    INSERT INTO `${BIGQUERY_PROJECT_ID}.${BIGQUERY_DATASET_ID}.${BIGQUERY_TABLE_NAME}`
    VALUES (NULL, 'memory', 0.00001, 'byte-second');'


# Join your resource prices with monitoring data

bq query --use_legacy_sql=false \
    'WITH
       billing_table AS (
         SELECT
           IFNULL(pricesheet.sku_id, "") AS sku_id,
           pricesheet.resource_name,
           pricesheet.rate,
           pricesheet.unit
         FROM
           `${BIGQUERY_PROJECT_ID}.${BIGQUERY_DATASET_ID}.${BIGQUERY_TABLE_NAME}` AS pricesheet
       ),
       request_based_cost_allocation AS (
       SELECT
         resource_usage.cluster_location,
         resource_usage.cluster_name,
         resource_usage.end_time,
         resource_usage.labels,
         resource_usage.namespace,
         resource_usage.resource_name,
         CASE
           WHEN resource_usage.resource_name = 'cpu' THEN 'CPU requested'
           WHEN resource_usage.resource_name = 'memory' THEN 'memory requested'
         END AS resource_name_with_type_and_unit,
         IFNULL(resource_usage.sku_id, "") AS sku_id,
         resource_usage.start_time,
         resource_usage.usage.amount AS amount,
         billing_table.rate * resource_usage.usage.amount AS cost,
         "request" AS type
       FROM
         `${BIGQUERY_PROJECT_ID}.${BIGQUERY_DATASET_ID}.gke_cluster_resource_usage` AS resource_usage
       INNER JOIN
         billing_table
       ON
         resource_usage.sku_id = billing_table.sku_id
         AND resource_usage.resource_name = billing_table.resource_name
       ),
       consumption_based_cost_allocation AS (
         SELECT
           resource_consumption.cluster_location,
           resource_consumption.cluster_name,
           resource_consumption.end_time,
           resource_consumption.labels,
           resource_consumption.namespace,
           resource_consumption.resource_name,
           CASE
             WHEN resource_consumption.resource_name = 'cpu' THEN 'CPU consumed'
             WHEN resource_consumption.resource_name = 'memory' THEN 'memory consumed'
           END AS resource_name_with_type_and_unit,
           IFNULL(resource_consumption.sku_id, "") AS sku_id,
           resource_consumption.start_time,
           resource_consumption.usage.amount AS amount,
           billing_table.rate * resource_consumption.usage.amount AS cost,
           "consumption" AS type
         FROM
           `${BIGQUERY_PROJECT_ID}.${BIGQUERY_DATASET_ID}.gke_cluster_resource_consumption` AS resource_consumption
         INNER JOIN
           billing_table
         ON
           resource_consumption.sku_id = billing_table.sku_id
           AND resource_consumption.resource_name = billing_table.resource_name
       )
       SELECT
         *
       FROM
         request_based_cost_allocation
       UNION ALL
       SELECT
         *
       FROM
         consumption_based_cost_allocation'
```

##### 6) Connect the DataStudio Anthos usage metering dashboard to BigQuery

Finally, go to [this URL](https://datastudio.google.com/c/u/0/reporting/d7374e12-54aa-438c-a9ab-dabcb90e83be) and copy a DataStudio dashboard for usage metering.