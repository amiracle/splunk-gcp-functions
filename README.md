# GCP Functions Library for Ingesting into Splunk

This library contains GCP Function Templates to read from:
- PubSub
- Metrics
- Google Cloud Storage
- Assets

[**PubSub Function**](https://github.com/amiracle/splunk-gcp-functions/blob/master/Examples/Example-1-PubSub.md)

This function pulls any event that is posted into PubSub and packages it up into a Splunk event. The event is then sent to the Http Event Collector (HEC). The function is written such that the event format can be sent compatible with Splunk's Add-On for Google Cloud Platform (https://splunkbase.splunk.com/app/3088/).
If any failures occur during sending the message to HEC, the event is posted back to a Pub-Sub Topic. A recovery function is provided which is executed via a Cloud Scheduler trigger (PubSub). The recovery function will attempt to clear out the PubSub retry topic and send these events into HEC.

**Metrics Function**
[Metrics Examples using Event Index](https://github.com/amiracle/splunk-gcp-functions/blob/master/Examples/Example-2a-Metrics.md)
[Metrics Examples using Metric Index](https://github.com/amiracle/splunk-gcp-functions/blob/master/Examples/Example-2b-Metrics.md)

This function is triggered by a Cloud Scheduler trigger (via PubSub). The function calls Stackdriver Monitoring APIs to retrieve the metrics (metrics request list, and poll frequency set in environment variable). These metrics are then sent to Splunk HEC. Two formats are supported - one to be compatible with the Add-on for GCP, sending the metrics as events into Splunk, the second is sent as a metric into Splunk's Metrics index.
As with the PubSub Function, any failed messages are sent into a PubSub topic for retry. A recovery function will attempt to resend periodically. 

[**Google Cloud Storage Examples**](https://github.com/amiracle/splunk-gcp-functions/blob/master/Examples/Example-3-GCS.md)

This function triggers on objects being written to a GCS Bucket. The bucket is set in when defining the function settings. All of the contents from the bucket is sent to HEC in batches of events – an event delimiter/breaker regex should be set in a function environment variable so that the function can break up the batches in the appropriate places. The batches are formed so that it can achieve an even event distribution across indexers (otherwise all of the content would go into 1 indexer). Code variables can be configured within the template to adjust the batch size. The Splunk HEC token should be created with the appropriate sourcetype for the events, as well as the index, as the events are not re-formatted/procecessed in any way.
Any messages that failed to be sent to HEC are sent into a PubSub topic for retry. A recovery function will attempt to resend periodically.

[**Assets**](https://github.com/amiracle/splunk-gcp-functions/tree/master/Assets)

This function is periodically triggered via a Cloud Scheduler push to a PubSub topic. The function calls GCP’s API to retrieve the cloud asset list. The list is written to a GCS bucket. (bucket location is defined in an environment variable).
Once in the GCS Bucket, the GCS function above can be used to read in the contents of the asset list into Splunk.

[**Alerts**](https://github.com/amiracle/splunk-gcp-functions/tree/master/Alert)

This function will can be triggered by a Stackdriver Alert (via webhook), sending the alert details to HEC. 

[**Retry Function**](https://github.com/amiracle/splunk-gcp-functions/tree/master/Retry)

This function periodically requests any failed events that were sent to a PubSub Topic, and re-tries sending to HEC. If there is a subsequent failure to sending to Splunk, the function will not acknowledge the pull from PubSub, and therefore will be re-tried at a later attempt.

## Getting started

The Functions library here has a set of pre-defined Examples for each of the Functions (contained in the Examples folder in this repo). The examples contains scripts that can be executed to create a sample function and its components such as PubSub topics. This is a great place to start to understand how these functions work, and how to set them up in your environment.

Note that the examples have a common retry function/configuration. This section will only need to be run once. In a similar way, the two metrics examples have a common trigger that can be reused for both examples.

To run the examples, you can either run directly from the Cloud Shell in the GCP console (click **>_** to activate Cloud Shell), or by downloading the SDK or Quickstart onto your host/local machine (see here - https://cloud.google.com/sdk/install)

Make sure you have installed git on the host running the example scripts (GCP's Cloud Shell already has this installed).

You will need a Splunk instance that has a public facing URL for HEC. To get up and running with Splunk, visit here https://www.splunk.com/en_us/software/splunk-cloud.html and set up a Splunk Cloud Instance.

(HEC can be configured using the instructions here - https://docs.splunk.com/Documentation/SplunkCloud/8.0.1/Data/UsetheHTTPEventCollector)

## Troubleshooting

For common issues with this library, head to the Examples folder and read the troubleshooting guide

## Dashboards

There is a template Splunk App available that provides dashboards to visualise the logs/metrics from these functions. This can be found/downloaded here - https://github.com/pauld-splunk/gcp_dashboards

## Terraform Automation

A Terraform Project for automating deployment of these functions is available here - https://github.com/nstonesplunk/cumulostrata
The project contains automation for configuring all parts of a data collection pipeline (Pub/Sub topics, schedulers, IAM service accounts, etc.) for GCS, Stackdriver Logging, and Stackdriver Metrics.

