# Prometheus Monitoring for Google Kubernetes Engine

## Introduction

In this hands-on tutorial, you'll setup a simple GKE cluster, then deploy Managed Service for Prometheus to ingest metrics from a simple application

### Pre-requisites
Select the project you'd like to work in:
<walkthrough-project-setup></walkthrough-project-setup>

Then, you'll deploy a standard GKE cluster, which will prompt you to authorize and enable the GKE API.

```bash
gcloud container clusters create gmp-cluster --num-nodes=1
```

Next you'll authenticate to the cluster

```bash
gcloud container clusters get-credentials gmp-cluster
```

Last you'll want to make sure the Monitoring API is enabled. This should be done automatically when creating the GKE cluster but you can make certain:

```bash
gcloud services enable monitoring.googleapis.com
```

On to the next section to start deployment!

## Deploy the Prometheus service
In this example, we'll walk through using the Managed Collection provided by Google Cloud to scrape metrics
 
https://cloud.google.com/stackdriver/docs/managed-prometheus/setup-managed

We'll first create a namespace to do the work in:

```bash
kubectl create ns gmp-test
```
Then we'll deploy the custom resource defintion and operator to deploy Prometheus within that GKE cluster

```bash
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.1.1/examples/setup.yaml
```

```bash
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.1.1/examples/operator.yaml
```


## Deploy the application

Next we'll deploy a really simple application which emits metrics at the /metrics endpoint:

```bash
kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/flask_deployment.yaml
```

```bash
kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/flask_service.yaml
```

We can check that this simple Python Flask app is serving metrics with the following command:
```bash
curl $(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}')/metrics
```

Then we'll tell Prometheus where to begin scraping the metrics from by applying the PodMonitoring file:

```bash
kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/prom_deploy.yaml
```

Before we finish up here, we'll generate some load on the application with a really simple interaction with the app:

```bash
timeout 120 bash -c -- 'while true; do curl $(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}'); sleep $((RANDOM % 4)) ; done'
```

This will run for 2 minutes, and when done, we can create a visualization of what this looks like!

## Observing the app via metrics

In this last section, we'll quickly use `gcloud` to deploy a custom monitoring dashboard that shows the metrics from this application in a line chart. Be sure to copy the entirety of this code block:

```bash
gcloud monitoring dashboards create --config='''
{
  "category": "CUSTOM",
  "displayName": "Prometheus Dashboard Example",
  "mosaicLayout": {
    "columns": 12,
    "tiles": [
      {
        "height": 4,
        "widget": {
          "title": "prometheus/flask_http_request_total/counter [MEAN]",
          "xyChart": {
            "chartOptions": {
              "mode": "COLOR"
            },
            "dataSets": [
              {
                "minAlignmentPeriod": "60s",
                "plotType": "LINE",
                "targetAxis": "Y1",
                "timeSeriesQuery": {
                  "apiSource": "DEFAULT_CLOUD",
                  "timeSeriesFilter": {
                    "aggregation": {
                      "alignmentPeriod": "60s",
                      "crossSeriesReducer": "REDUCE_NONE",
                      "perSeriesAligner": "ALIGN_RATE"
                    },
                    "filter": "metric.type=\"prometheus.googleapis.com/flask_http_request_total/counter\" resource.type=\"prometheus_target\"",
                    "secondaryAggregation": {
                      "alignmentPeriod": "60s",
                      "crossSeriesReducer": "REDUCE_MEAN",
                      "groupByFields": [
                        "metric.label.\"status\""
                      ],
                      "perSeriesAligner": "ALIGN_MEAN"
                    }
                  }
                }
              }
            ],
            "thresholds": [],
            "timeshiftDuration": "0s",
            "yAxis": {
              "label": "y1Axis",
              "scale": "LINEAR"
            }
          }
        },
        "width": 6,
        "xPos": 0,
        "yPos": 0
      }
    ]
  }
}
'''
```

Once created, navigate to `Monitoring > Dashboards` to see the newly created `Prometheus Dashboard Example` -- click through below to see how to get there.
<walkthrough-menu-navigation sectionId="MONITORING_SECTION;stackdriver_dashboards"></walkthrough-menu-navigation>

## Conclusion

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

Congratulations! You've completed this tutorial and have seen the basics of deploying a GKE app with Prometheus Metrics, and creating a Cloud Monitoring Dashboard from it.
<walkthrough-inline-feedback></walkthrough-inline-feedback>

If you'd like to learn more, check out our documentation for [deploying self-managed collection](https://cloud.google.com/stackdriver/docs/managed-prometheus/setup-unmanaged)
