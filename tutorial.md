# Prometheus Monitoring for Google Kubernetes Engine

## Introduction

In this hands-on tutorial, you'll setup a simple GKE cluster, then deploy Managed Service for Prometheus to ingest metrics from a simple application

### Pre-requisites
Select the project you'd like to work in:
<walkthrough-project-setup></walkthrough-project-setup>

Then, you'll deploy a standard GKE cluster, which will prompt you to authorize and enable the GKE API.

```
gcloud container clusters create gmp-cluster --num-nodes=1
```

Next you'll authenticate to the cluster
```
gcloud container clusters get-credentials gmp-cluster
```

Last you'll want to make sure the Monitoring API is enabled. This should be done automatically when creating the GKE cluster but you can make certain:

```
gcloud services enable monitoring.googleapis.com
```

On to the next section to start deployment!

### Deploy the Prometheus service
In this example, we'll walk through using the Managed Collection provided by Google Cloud to scrape metrics
 
https://cloud.google.com/stackdriver/docs/managed-prometheus/setup-managed


### Deploy the application

More instructions.

### Observing the app via metrics

## Conclusion

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

We're done!

Here’s what to do next:

(Follow up content)



---------- working area

timeout 120 bash -c -- 'while true; do curl IP_ADDR; sleep $((RANDOM % 4)) ; done'