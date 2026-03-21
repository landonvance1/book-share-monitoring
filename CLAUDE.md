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
- `booksharing-api:8080/metrics` (book-share-api, 10s interval)
- `cover-detection:8000/metrics` (cover-detection, 15s interval)

**Grafana** (`grafana/grafana:11.1.0`) — dashboards and visualization, exposed on port 3100. Datasource and dashboard provisioning are file-based under `grafana/provisioning/`.

### File Structure

```
docker-compose.yml          # Prometheus + Grafana services
.env.example                # Grafana admin credentials template
prometheus/
  prometheus.yml            # Scrape configuration
grafana/
  provisioning/
    datasources/
      datasource.yml        # Prometheus datasource
    dashboards/
      dashboard.yml         # Dashboard provider config
  dashboards/
    bookshare-overview.json # Overview dashboard (health, request rate, latency, errors)
```

### Metric Name Conventions

The API uses OpenTelemetry metric names (`http_server_request_duration_seconds_*`). Cover-detection uses prometheus-fastapi-instrumentator names (`http_request_duration_seconds_*`). Dashboard queries account for both naming schemes.
