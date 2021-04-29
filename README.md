# Prometheus Example App

This example app serves as an example of how one can easily instrument HTTP handlers with [Prometheus][prometheus] metrics. It uses the Prometheus [go client][client-golang] to create a new Prometheus registry.

Usage is simple, on any request to `/` the request will result in a `200` response code. This increments the counter for this response code. Similarly the `/err` endpoint will result in a `404` response code, therefore increments that respective counter. Duration metrics are also exposed for any request to `/`. The `/403` endpoint will result in a `403` response code, the `/500` endpoint will result in a `500` response code, the `/502` endpoint will result in a `502` response code.

A Docker image is available at: `quay.io/chiaretto/prometheus-example-app:latest`

## Deploying in a Kubernetes cluster

First, deploy one instance of this example application, which listens and exposes metrics on port 8080 using the following [Deployment manifest](manifests/deployment.yaml).

Then, in order for Prometheus to be able to discover and scrape these metrics, in this repository we provide [Pod Monitor][prometheus-operator-crd] custom resource as an example. For that, [Prometheus Operator][prometheus-operator] needs to be running in the cluster. To install it, follow this guide: [quickstart][prometheus-operator-quickstart].

For this example application, [PodMonitor manifest](manifests/pod-monitor.yaml) describes how the metrics can be discovered and scrapped by Prometheus.

[prometheus]:https://prometheus.io/
[client-golang]:https://github.com/prometheus/client_golang
[prometheus-operator]:https://github.com/prometheus-operator/prometheus-operator
[prometheus-operator-quickstart]:https://github.com/coreos/prometheus-operator#quickstart
[prometheus-operator-crd]:https://github.com/coreos/prometheus-operator#customresourcedefinitions

## Exposed Prometheus metrics

The following metrics are exposed:

- `version` - of type _gauge_ - containing the app version - as a constant metric value `1` and label `version`, representing this app version
- `http_requests_total` - of type _counter_ - representing the total numbere of incoming HTTP requests
- `http_request_duration_seconds` - of type _histogram_, representing duration of all HTTP requests
- `http_request_duration_seconds_count`- total count of all incoming HTTP requeests
- `http_request_duration_seconds_sum` - total duration in seconds of all incoming HTTP requests
- `http_request_duration_seconds_bucket` - a histogram representation of the duration of the incoming HTTP requests

The sample output of the `/metric` endpoint after 5 incoming HTTP requests shown below.

Note: with no initial incoming request, only `version` metric is reported.

```
# HELP http_request_duration_seconds Duration of all HTTP requests
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="0.005"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="0.01"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="0.025"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="0.05"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="0.1"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="0.25"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="0.5"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="1"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="2.5"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="5"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="10"} 2
http_request_duration_seconds_bucket{code="200",handler="found",method="get",le="+Inf"} 2
http_request_duration_seconds_sum{code="200",handler="found",method="get"} 5.9709e-05
http_request_duration_seconds_count{code="200",handler="found",method="get"} 2
# HELP http_requests_total Count of all HTTP requests
# TYPE http_requests_total counter
http_requests_total{code="200",method="get"} 2
http_requests_total{code="403",method="get"} 1
http_requests_total{code="500",method="get"} 5
http_requests_total{code="502",method="get"} 3
# HELP version Version information about this binary
# TYPE version gauge
version{version="v0.4.0"} 1
```
