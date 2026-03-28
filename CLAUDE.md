# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

book-share-monitoring provides monitoring, alerting, and observability for the BookShare platform. It tracks the health and performance of two services:

### Monitored Services

**book-share-api** (ASP.NET Core 8.0)
- Community-based book lending platform backend
- PostgreSQL 15 database, JWT auth, SignalR real-time messaging
- Manages the full share lifecycle: Requested -> Ready -> PickedUp -> Returned -> HomeSafe
- REST endpoints for auth, books, user libraries, shares, chat, notifications, communities

**book-share-cover-detection** (Python microservice)
- Analyzes book cover photos to extract title/author using local ML models
- Florence-2 (OCR) + GLiNER (NLP), with PyTorch and ONNX runtime options
- Endpoints: `POST /analyze` (image analysis), `GET /health` (status)

## Infrastructure

The monitoring stack runs via Docker Compose on the external `booksharing` network (shared with the services being monitored).

**Prometheus** (`prom/prometheus:v2.53.0`) — scrapes metrics from services by container name:
- `booksharing-api:8081/metrics` (book-share-api, 10s interval)
- `cover-detection:8000/metrics` (cover-detection, 15s interval)

**Grafana** (`grafana/grafana:11.1.0`) — dashboards and visualization, exposed on port 3100. Datasource and dashboard provisioning are file-based under `grafana/provisioning/`.

**Loki** (`grafana/loki:3.6.0`) — log aggregation and storage. Not exposed to the host; only reachable on the internal `monitoring-internal` Docker network by Grafana and Alloy. Stores logs on the local filesystem with 7-day retention.

**Alloy** (`grafana/alloy:v1.14.1`) — log collector that discovers Docker containers via the Docker socket, parses JSON logs from both services using service-aware `stage.match` blocks, and forwards to Loki. Replaces the now-EOL Promtail. Runs as root (required for Docker socket access).

For a full diagram of containers, networks, and ports see `docs/network-diagram.md`.

### File Structure

```
docker-compose.yml          # Prometheus + Grafana + Loki + Alloy services
.env.example                # Grafana admin credentials template
prometheus/
  prometheus.yml            # Scrape configuration
loki/
  loki-config.yml           # Loki storage, retention, and schema config
alloy/
  config.alloy              # Alloy log collection pipeline (discovery, relabel, JSON parse, forward)
grafana/
  provisioning/
    datasources/
      datasource.yml        # Prometheus + Loki datasources
    dashboards/
      dashboard.yml         # Dashboard provider config
  dashboards/
    bookshare-overview.json # Overview dashboard (health, request rate, latency, errors, cover detection stage latency)
    bookshare-logs.json     # Log exploration dashboard (log volume, level/category filters, API error rate, per-service logs)
```

### Log Label Conventions

All containers get `service` and `container` labels from Alloy's relabel rules. Both services additionally get `level` and `category` as indexed labels, parsed from their JSON log output via service-aware `stage.match` blocks in `alloy/config.alloy`:

- **book-share-api**: `LogLevel` → `level` (e.g. `Information`, `Warning`, `Error`), `Category` → `category` (e.g. `Services.ShareService`, `Hubs.ChatHub`)
- **cover-detection**: `level` → `level` (e.g. `INFO`, `ERROR`), `logger_name` → `category` (e.g. `app.services.analyzer`, `app.engines.florence2_onnx_engine`)

### Metric Name Conventions

The API uses OpenTelemetry metric names (`http_server_request_duration_seconds_*`). Cover-detection uses prometheus-fastapi-instrumentator names (`http_request_duration_seconds_*`). Cover-detection also exposes custom stage histograms: `cover_detection_ocr_duration_seconds`, `cover_detection_nlp_duration_seconds`, and `cover_detection_total_duration_seconds`. Dashboard queries account for all of these naming schemes.
