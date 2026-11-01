# Echo — Crypto Narrative Pulse Tracker

<p>
  <img src="https://img.shields.io/badge/Project-Streaming%20Analytics-2563EB?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Tests-153%20reported-2E8B57?style=for-the-badge&logo=pytest&logoColor=white"/>
  <img src="https://img.shields.io/badge/Python-3.11%2B-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
</p>

<p>
  <a href="https://skillicons.dev"><img src="https://skillicons.dev/icons?i=python,flask,nextjs,docker,sqlite&perline=5" alt="Echo technology stack"/></a>
</p>

A streaming analytics project that turns social signals into a compact view of emerging crypto narratives. It combines message ingestion, sentiment analysis, phrase clustering, influencer signals, and price/sentiment divergence into a live dashboard and alerting workflow.

## ✨ What it does

- Ingests social messages through a webhook and processes them through a Pathway-based pipeline.
- Scores sentiment with VADER and a crypto-oriented lexicon.
- Groups recurring phrases to surface developing narratives.
- Combines sentiment, phrase momentum, and influencer signals into a pulse score.
- Compares narrative sentiment with market price movement to identify divergence.
- Exposes metrics over HTTP and WebSockets, with optional Telegram alerts.
- Includes a simulator for repeatable demonstrations without a live message source.

## 🏗️ Architecture

```mermaid
flowchart LR
    S[Social signal webhook] --> P[Pathway pipeline]
    M[Demo simulator] --> S
    P --> N[Sentiment and phrase analysis]
    N --> C[Pulse score and divergence]
    C --> A[Flask API and WebSockets]
    A --> D[Dashboard]
    C --> T[Optional Telegram alerts]
    A --> R[Optional RAG query]
```

## 🧰 Stack

Python · Pathway · Flask · VADER · WebSockets · Next.js · Ollama (optional) · Telegram Bot API

## 🚀 Run with Docker

### Prerequisites

- Docker Engine
- Docker Compose

### Start the pipeline

```bash
git clone https://github.com/SSRINIBASS/echo.git
cd echo
docker compose up --build pipeline
```

The API and webhook service are exposed on ports `8000` and `8080` by default.

### Run the demo simulator

In another terminal:

```bash
docker compose --profile demo run --rm simulator
```

### Start the dashboard and Telegram bot

Configure the required environment variables in a local `.env` file, then run:

```bash
docker compose --profile full up --build
```

### Optional local LLM

```bash
docker compose --profile llm up -d ollama
```

## 🧪 Tests

Run the Python test suite:

```bash
python -m pytest tests/ -v
```

The repository README reports 153 tests across core pipeline, sentiment, pulse score, phrase clustering, influencer tracking, and performance monitoring.

## 🔐 Notes

The full Pathway pipeline is intended to run in the Linux container environment. A local simulator/demo path is provided for development and walkthroughs. External API credentials belong in local environment configuration, never in committed files.
