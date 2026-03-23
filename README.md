# book-share-monitoring

Monitoring and observability solution for the BookShare platform — a community-based book lending system.

## Purpose

This repository provides monitoring, alerting, and observability tooling for the BookShare ecosystem:

- **[book-share-api](https://github.com/landonvance1/book-share-api)** — ASP.NET Core 8.0 backend API powering community-based book lending. Handles the full borrowing lifecycle (discovery, request, pickup, return), real-time chat via SignalR, JWT authentication, and PostgreSQL persistence.
- **[book-share-cover-detection](https://github.com/landonvance1/book-share-cover-detection)** — Python microservice that analyzes book cover photographs to extract title and author information using locally-running ML models (Florence-2 for OCR, GLiNER for NLP). Supports both PyTorch and ONNX runtimes.

## Monitored Services

| Service | Stack | Key Endpoints |
|---------|-------|---------------|
| book-share-api | ASP.NET Core 8.0, PostgreSQL 15, SignalR | REST API (auth, books, shares, chat, notifications, communities) |
| book-share-cover-detection | Python, Florence-2, GLiNER | `POST /analyze`, `GET /health` |

## Stack

| Component | Image | Purpose |
|-----------|-------|---------|
| Prometheus | `prom/prometheus:v2.53.0` | Metrics scraping and storage |
| Grafana | `grafana/grafana:11.1.0` | Dashboards and visualization |
| Loki | `grafana/loki:3.6.0` | Log aggregation and storage |
| Alloy | `grafana/alloy:v1.14.1` | Log collection from Docker containers |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- The `booksharing` Docker network (shared with the monitored services):
  ```bash
  docker network create booksharing
  ```

## Quick Start

1. Copy the example environment file and adjust as needed:
   ```bash
   cp .env.example .env
   ```

2. Start the monitoring stack:
   ```bash
   docker compose up -d
   ```

3. Access the services:
   - **Grafana**: http://localhost:3100 (login with credentials from your `.env`)
   - **Prometheus**: http://localhost:9090

Grafana is provisioned with two dashboards under the **BookShare** folder:
- **BookShare Overview** — service health, request rate, latency, and error rate (Prometheus)
- **BookShare Logs** — log volume and per-service log explorer (Loki)

Scrape targets and log sources will show as down until the monitored services are running on the `booksharing` network.

## Adding Scrape Targets

To monitor a new service, add a scrape job to `prometheus/prometheus.yml`:

```yaml
- job_name: my-service
  scrape_interval: 15s
  metrics_path: /metrics
  static_configs:
    - targets:
        - my-container-name:port
```

Then restart Prometheus:

```bash
docker compose restart prometheus
```

## Logs

Alloy automatically discovers and collects logs from all Docker containers on the host. Logs are queryable in Grafana using [LogQL](https://grafana.com/docs/loki/latest/query/) via the Loki datasource.

Useful label filters:
- `{service="book-share-api"}` — API logs
- `{service="cover-detection"}` — cover detection logs
- `{container="<name>"}` — logs for any specific container by its Docker name
