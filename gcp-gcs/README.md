---
title: "GCP Storage"
date: "2025-01-30"
coverImage: "icon_cloud_192pt_clr-1.png"
---

> **[NOTICE]** This function is deprecated in favor of the [gcp-logs](https://coralogix.com/docs/integrations/gcp/gcp-logs/) pull integration. Please use the new integration for all new GCP logs integrations. [terraform-provider-coralogix](https://github.com/coralogix/terraform-provider-coralogix) provides the `coralogix_integration` resource for this.

Coralogix provides a predefined function to forward your logs from Google Cloud Platform (GCP) Storage straight to Coralogix.

## Setup

There are two methods to deploy this integration. The first is using gcloud cli, and the second is to use our Terraform module.

For the gcloud cli method, you will need to select the coralogix domain endpoint that corresponds to your Coralogix AWS region from the table below. You will also need a service account with the necessary permissions to invoke the function, access storage buckets and objects, and receive Eventarc events. Check the Service Account Setup section below for example steps to create the service account and custom role.

### Coralogix Legacy Logs API

| Coralogix Domain | Coralogix AWS Region | Endpoint |
| --- | --- | --- |
| coralogix.com | eu-west-1<br/>[EU1 - Ireland] | https://ingress.coralogix.com/api/v1/logs |
| coralogix.in | ap-south1<br/>[AP1 - India] | https://ingress.coralogix.in/api/v1/logs |
| coralogix.us | us-east2<br/>[US1 - Ohio] | https://ingress.coralogix.us/api/v1/logs |
| eu2.coralogix.com | eu-north-1<br/>[EU2 - Stockholm] | https://ingress.eu2.coralogix.com/api/v1/logs |
| coralogixsg.com | ap-southeast-1<br/>[AP2 - Singapore] | https://ingress.coralogixsg.com/api/v1/logs |
| cx498.coralogix.com | us-west-2<br/>[US2 - Oregon] | https://ingress.cx498.coralogix.com/api/v1/logs |
| ap3.coralogix.com | ap-southeast-3<br/>[AP3 - Jakarta] | https://ingress.ap3.coralogix.com/api/v1/logs |

For example, if your top level domain is coralogix.us, use the following as your endpoints:

``` bash
CORALOGIX_LOG_URL=https://ingress.coralogix.us/api/v1/logs
CORALOGIX_TIME_DELTA_URL=https://ingress.coralogix.us/sdk/v1/time
```

### gcloud CLI

When using the gcloud CLI, you'll first need to download the code files from our repository. An example using curl is provided below:

``` bash
mkdir -p gcsToCoralogix
cd gcsToCoralogix
curl -sSL -O https://raw.githubusercontent.com/coralogix/terraform-coralogix-google/master/modules/storage/src/{main.py,requirements.txt}
```

After you have the code files in gcsToCoralogix folder, deploy a cloud function with the below command. Make sure you update `YOUR_GCP_PROJECT_ID` with your project ID, your GCP region, the `YOUR_BUCKET_NAME` trigger resource, and the values in `set-env-vars` accordingly. None of the values require quotation marks.

``` bash
gcloud functions deploy gcsToCoralogix \
--project=YOUR_GCP_PROJECT_ID \
--region=us-central1 \
--runtime=python38 \
--memory=1024MB \
--timeout=60s \
--entry-point=to_coralogix \
--source=gcsToCoralogix \
--trigger-resource=YOUR_BUCKET_NAME \
--trigger-event=google.storage.object.finalize \
--service-account=YOUR_SERVICE_ACCOUNT_EMAIL \
--set-env-vars="private_key=YOUR_PRIVATE_KEY,app_name=APP_NAME,sub_name=SUB_NAME,CORALOGIX_LOG_URL=https://ingress.coralogix.com/api/v1/logs,CORALOGIX_TIME_DELTA_URL=https://ingress.coralogix.com/sdk/v1/time"
```

After deploying, double check your Google Cloud console to validate the Cloud Function was deployed as expected.

### Terraform

[Here](https://github.com/coralogix/terraform-coralogix-google/tree/master/examples/storage) is the Terraform module to deploy the Cloud Function.

**NOTE** - The Terraform module has not yet been updated to support Gen2 Cloud Functions or the use of a custom service account. Presently the TF deployment cloud function is "public".

Add these modules to your manifest and change its options:

``` tf
provider "google" {
project = "YOUR_GCP_PROJECT_ID"
region = "us-central1"
}

module "storage" {
  source = "coralogix/google/coralogix//modules/storage"

  coralogix_region = "Europe"
  private_key      = "YOUR_API_KEY"
  application_name = "GCP"
  subsystem_name   = "GCS"
  bucket           = "YOUR_BUCKET_NAME"
}
```

Initialize the module and apply these changes:

``` bash
terraform init
terraform apply
```

### Service Account Setup

We need a service account with the necessary permissions to invoke the function, access storage buckets, and receive Eventarc events. Below are two options for creating the service account and custom role. This Service Account can be shared by multiple deployments within your project, so it is managed outside of the integration deployment itself.

#### Option 1: Using gcloud CLI

```bash
# Create the service account
gcloud iam service-accounts create gcs-to-coralogix \
    --description="Service account for GCS to Coralogix integration" \
    --display-name="GCS to Coralogix SA"

# Get your project ID
PROJECT_ID=$(gcloud config get-value project)

# Create custom role with all needed permissions
gcloud iam roles create gcsToCoralogixRole \
    --project=$PROJECT_ID \
    --title="GCS to Coralogix Role" \
    --description="Custom role for GCS to Coralogix integration" \
    --permissions=storage.buckets.get,storage.objects.get,run.routes.invoke,eventarc.events.receiveEvent

# Grant the custom role to the service account
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:gcs-to-coralogix@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="projects/$PROJECT_ID/roles/gcsToCoralogixRole"
```

#### Option 2: Using Terraform

```tf
# Variables
variable "project_id" {
  description = "The GCP project ID"
  type        = string
}

# Service Account
resource "google_service_account" "gcs_to_coralogix" {
  account_id   = "gcs-to-coralogix"
  display_name = "GCS to Coralogix SA"
  description  = "Service account for GCS to Coralogix integration"
  project      = var.project_id
}

# Custom IAM Role
resource "google_project_iam_custom_role" "gcs_to_coralogix_role" {
  role_id     = "gcsToCoralogixRole"
  title       = "GCS to Coralogix Role"
  description = "Custom role for GCS to Coralogix integration"
  permissions = [
    "storage.buckets.get",
    "storage.objects.get",
    "run.routes.invoke",
    "eventarc.events.receiveEvent"
  ]
  project = var.project_id
}

# IAM binding
resource "google_project_iam_binding" "gcs_to_coralogix_binding" {
  project = var.project_id
  role    = google_project_iam_custom_role.gcs_to_coralogix_role.id
  members = [
    "serviceAccount:${google_service_account.gcs_to_coralogix.email}"
  ]
}

# Outputs
output "service_account_email" {
  description = "The email address of the service account"
  value       = google_service_account.gcs_to_coralogix.email
}

output "custom_role_id" {
  description = "The ID of the custom role"
  value       = google_project_iam_custom_role.gcs_to_coralogix_role.id
}
```
Initialize the module and apply these changes:

``` bash
terraform init
terraform apply
```
Capture the Service Account email and Custom Role ID from the outputs for use in the deployment.
## **Support**

**Need help?**

Our world-class customer success team is available 24/7 to walk you through your setup and answer any questions that may come up.

Feel free to reach out to us **via our in-app chat** or by sending us an email at [support@coralogix.com](mailto:support@coralogix.com).
