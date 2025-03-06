# Prometheus Exporter for CF Stack Usage

Looking to gain insights into distribution of stacks throughout a CF
installation? This exporter scrapes the CF API to generate a metric called
`cf_stack_count` with the label of `stack` to identify the count of applications
using each stack installed on a CF installation.

It's meant to be run as a cf app, but can easily run wherever you've got Python.

## Installation

Take a look at `manifest.yml`: you'll need three bits of information:

`CF_API_URL` is the full URL for your CF's API (i.e. `https://api.sys.cloud.seventhprotocol.com`)  
`CF_USERNAME` is an UAA account that has at minimum read-only permissions to all of the `/v3/apps` endpoint  
`CF_PASSWORD` is the password to the account above.  

Optionally, you can also configure:  
`SCRAPE_INTERVAL` which dictates how often the background task of refreshing the metric occurs. By default it's 300 seconds.  
`LOG_LEVEL` which controls verbosity for debugging purposes. By default it's INFO (pretty quiet)  
`SKIP_SSL_VERIFY` for whether or not to skip SSL validation. By default it's False (validates SSL)  
`INCLUDE_INVALID_STACKS` to control whether or not we report stacks that are invalid (i.e typos from developers)  


An example CF push:

```
cf push \
--var cf_api_url=https://api.sys.cloud.seventhprotocol.com \
--var cf_username=readonly \
--var cf_password=deepbluesea
```

If your CF installation doesn't allow downloading dependencies from the internet
(common), run this command to vendor the dependencies prior to pushing: 

```
pip3 download \
--only-binary=:all: \
--python-version=3.12 \
--platform=manylinux2014_x86_64 \
--dest=vendor \
--requirement=requirements.txt
```

## Adding to Healthwatch

Running a Tanzu Application Service installation, with Healthwatch 2.0? To add
the scrape job to HW, go to your OpsMan, then the Healthwatch tile. Navigate to
Prometheus > Additional Scrape Jobs.

Paste the following into the scrape job configuration:
```
job_name: 'stack-usage'
scheme: https
scrape_interval: 60s
static_configs:
- targets: [ 'stack-usage-prometheus-exporter.apps.cloud.seventhprotocol.com' ]
  labels:
    env: 'staging'
```

Replace the `target` with the URL of your exporter instance, and while the label
isn't required, it's recommended to add your environment name to it (i.e.
sandbox, staging, production)