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
