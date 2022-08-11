# GCP log exporter

This repo contains an example of code to use with code functions to export logs from buckets to Coralogix using cloud functions.

## Installation manual
Coralogix documentations can be found [Here](https://coralogix.com/integrations/gcp-log-explorer/)

## Configuration
### A few basic environment variables are required here:
- private_key - Your secret taken from your Coralogix UI.
- app_name - Your application name, this is how you can label your pplications logs.
- sub_name - Your subsystem name, this is a second label for your applications logs.

If your Coralogix account top level domain is different than ‘.com’ add the following environment variable, 
```
CORALOGIX_LOG_URL=https://api.<Cluster URL>/api/v1/logs
```

Requirements:
-------------
* A Coralogix account
* Installation of GCP CLI (gcloud)
* A GCP cloud storage configured.
* Permissions to configure a function.

To setup the function, execute this:

.. code-block:: bash

	#First lets clone the repository
	$ git clone https://github.com/coralogix/coralogix-gcp-serverless.git &&
    	$ cd coralogix-gcp-serverless
    
	$ gcloud functions deploy gcp-log-exporter \
		--project=<YOUR_GCP_PROJECT_ID> \
		--region=<GCP_REGION_NAME> \
		--runtime=python39 \
		--memory=1024MB \
		--timeout=300s \
		--entry-point=to_coralogix \
		--trigger-bucket=<YOUR_STORAGE_BUCKET> \
		--source=gcp-log-exporter \
		--set-env-vars="private_key=<YOUR_PRIVATE_KEY>,app_name=<APP_NAME>,sub_name=<SUB_NAME>"
	# additional variables available and their defaults: 'newline_pattern=/(?:\r\n|\r|\n)/g', 'sampling=1', 'CORALOGIX_URL=api.coralogix.com'