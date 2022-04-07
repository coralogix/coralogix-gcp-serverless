Google Cloud Pub/Sub
====================

*Coralogix* provides a predefined function to forward your logs from ``Google Cloud Pub/Sub`` straight to *Coralogix*.

Setup
-----

gcloud CLI
~~~~~~~~~~

To setup the function, execute this:

.. code-block:: bash
	#First lets clone the repository
	git clone https://github.com/coralogix/coralogix-gcp-serverless.git &&
    	cd coralogix-gcp-serverles
    
	gcloud functions deploy gcp-pub-sub \
		--project=<YOUR_GCP_PROJECT_ID> \
		--region=<GCP_REGION_NAME> \
		--runtime=python38 \
		--memory=1024MB \
		--timeout=60s \
		--entry-point=to_coralogix \
		--source=gcp-pub-sub \
		--trigger-resource=<YOUR_PUBSUB_TOPIC_NAME> \
		--trigger-event=google.pubsub.topic.publish \
		--set-env-vars="private_key=<YOUR_PRIVATE_KEY>,app_name=<APP_NAME>,sub_name=<SUB_NAME>"
