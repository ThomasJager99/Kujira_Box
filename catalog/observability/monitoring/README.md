# Monitoring Stack: Prometheus + Grafana + Node Exporter + cAdvisor

<p align="center">
  <img src="../../../assets/logos/grafana.svg" width="180">
</p>

<br>

<p align="center">
  <img src="../../../assets/logos/prometheus.svg" width="180">
</p>

<br>

<p align="center">
  <img src="../../../assets/logos/cadvisor.png" width="180">
</p>

<br>


This directory contains an example deployment of a monitoring and logging
stack used to observe containerized infrastructure.

The stack collects system metrics, container metrics and logs, and provides
a centralized interface for visualization and analysis.

## Components

### Metrics collection

Prometheus is used as the main metrics collection and storage system.

Metrics are gathered from:

- Node Exporter — host system metrics
- cAdvisor — container metrics

Prometheus periodically scrapes these exporters and stores time-series data.

### Logging

A centralized logging pipeline is provided using:

- Loki — log storage system
- Promtail — log collector

Promtail reads container logs from the Docker host and forwards them to Loki.
Grafana can then query Loki to visualize logs alongside metrics.

### Visualization

Grafana provides dashboards and query tools for both metrics and logs.

Grafana connects to:

- Prometheus (metrics)
- Loki (logs)

This allows infrastructure monitoring and log inspection from a single
interface.

## Stack Architecture

Node Exporter ─┐
│
cAdvisor ──────┤
▼
Prometheus
│
▼
Grafana

Docker logs
│
▼
Promtail
│
▼
Loki
│
▼
Grafana

## Deployment Notes

- Services run inside the same Docker network
- Logging configuration is standardized using Docker logging driver limits
- Health checks are used to ensure service availability
- Resource limits are applied through environment variables

## Example Configuration

Deployment configuration is available in:

docker-compose.yml.exmpl
.env.exmpl

These files demonstrate a reproducible monitoring stack that can be deployed
using Docker Compose.

## Thanks

Thanks to the maintainers of the following projects:

- Prometheus
- Grafana
- Loki
- Node Exporter
- cAdvisor

for providing powerful open-source observability tools.
