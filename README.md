# GCP Log Forwarder


## Creating a GCP Project, Service Account, and Assigning Roles via CLI

Here's how to create a GCP project, a service account, and assign it the necessary roles to create and read logs, along with instructions on downloading the service account key, all using the `gcloud` CLI:

**1. Create a GCP Project**

```bash
PROJECT_ID=loggin
gcloud projects create $PROJECT_ID --set-as-default
```

Replace `PROJECT_ID` value with your desired project ID. The `--set-as-default` flag sets this new project as your default for subsequent commands.

**2. Create a Service Account**

```bash
SERVICE_ACCOUNT_NAME=loggingforwarder
gcloud iam service-accounts create $SERVICE_ACCOUNT_NAME --display-name "SERVICE_ACCOUNT_DISPLAY_NAME"
```

Replace `SERVICE_ACCOUNT_NAME` value with a unique name for your service account and `SERVICE_ACCOUNT_DISPLAY_NAME` with a human-readable name.

**3. Assign Roles to the Service Account**

```bash
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member "serviceAccount:${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role "roles/logging.logWriter"

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member "serviceAccount:${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role "roles/logging.viewer"
```

Make sure to replace `PROJECT_ID` and `SERVICE_ACCOUNT_NAME` with your actual values.

**4. Download the Service Account Key**

```bash
gcloud iam service-accounts keys create key.json \
  --iam-account ${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
```

This command will create a service account key file named `key.json` in your current directory. Again, replace `SERVICE_ACCOUNT_NAME` and `PROJECT_ID` accordingly.

**Important Notes:**

* **Project ID availability:** Ensure your chosen `PROJECT_ID` is unique and follows GCP's naming conventions.
* **gcloud CLI:** Make sure you have the gcloud CLI installed and configured with the correct credentials.
* **Security:** Keep the `key.json` file secure and treat it like a password.
* **Regional services:** If you're using regional services like Cloud Logging, you might need to adjust the roles and commands to be region-specific.


This markdown version provides a clean and organized way to present the CLI commands for managing your GCP resources. You can easily copy and paste these commands into your terminal for execution.


## Deploy to kubernetes (on-prem)

Note: Login to your kubenretes cluster we are going to need `kubectl` and `helm`client so install them.


### Set your values
Edit `logger-values.yaml` and change at least the values of:

```
k8s_cluster_name NAMEOFTHECLUSTER
k8s_cluster_location  LOCATIONOFTHECLUSTER
```

### Add helm repos

```bash
helm repo add fluent https://fluent.github.io/helm-charts
helm repo add deliveryhero https://charts.deliveryhero.io
```

### Create Namespace

```bash
export NAMESPACE='logging'
kubectl create ns $NAMESPACE
```
### Create Google Cloud Secret

```bash
kubectl create secret generic google-service-credentials \
--from-file key.json -n $NAMESPACE
```

### Deploy the log forwarder

```bash
helm install log-forwarder fluent/fluent-bit \
-n $NAMESPACE -f logger-values.yaml
```


### Deploy the cluster event forwarder

```bash
helm install event-forwarder deliveryhero/k8s-event-logger \
-n $NAMESPACE
```# gcp-log-forwarder
