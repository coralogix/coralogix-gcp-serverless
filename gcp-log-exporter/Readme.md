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

### And general configs
- Entry Point - To_coralogix
- Runtime - Python3.8
