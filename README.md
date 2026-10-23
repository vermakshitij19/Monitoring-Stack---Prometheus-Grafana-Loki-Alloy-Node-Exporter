# Monitoring
# Author: Kshitij Verma
This directory contains Helm values for the cluster monitoring and log-collection stack:

- `monitoring-values.yaml` configures Grafana persistence, Prometheus retention and storage, and Alertmanager storage for the Prometheus Community `kube-prometheus-stack` chart.
- `loki-values.yaml` configures Loki as a single-replica, filesystem-backed monolith.
- `alloy-values.yaml` configures Grafana Alloy as a DaemonSet that collects Kubernetes pod logs and forwards them to Loki.

## Prerequisites

- A Kubernetes cluster with `kubectl` configured to use it.
- Helm 3.
- A cluster storage class that can provision persistent volumes.

## Install

Add the chart repositories and create the monitoring namespace:

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -
```

Install Prometheus and Loki, then install Alloy after Loki is available:

```sh
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --values monitoring-values.yaml

helm upgrade --install loki grafana/loki \
  --namespace monitoring \
  --values loki-values.yaml

helm upgrade --install alloy grafana/alloy \
  --namespace monitoring \
  --values alloy-values.yaml
```

Alloy sends pod logs to `http://loki-gateway.monitoring.svc.cluster.local/loki/api/v1/push`.
The Loki gateway service must be reachable at that address for log delivery to work.

## Access Prometheus and Grafana locally

Forward the service ports from a terminal:

```sh
kubectl port-forward --namespace monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
```

Open Prometheus at `http://localhost:9090`. In another terminal, forward Grafana:

```sh
kubectl port-forward --namespace monitoring svc/prometheus-grafana 3000:80
```

Open Grafana at `http://localhost:3000`. Use the credentials configured for the Helm release; do not commit credentials or secret overrides to this directory.

## Files

- `PortNumbers.png` and `PromQL+Intro.xlsx` are reference materials.
- Update the values files to change persistence, retention, or collection settings before applying a Helm release.
