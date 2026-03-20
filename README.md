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
