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
   - **Prometheus**: http://localhost:9090
   - **Grafana**: http://localhost:3100 (login with credentials from your `.env`)

Scrape targets will show as down until the monitored services are running with metrics endpoints enabled.

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
