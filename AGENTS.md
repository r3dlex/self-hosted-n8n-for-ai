# AGENTS.md — Project Governance

This document is the single source of truth for project conventions, architecture decisions, and contribution rules. All other governance documents (CLAUDE.md, CONTRIBUTING.md) refer here.

## Project purpose

A zero-install, Docker Compose based self-hosted n8n setup with PostgreSQL. Designed for learning and experimentation with AI workflows, not production use.

## Architecture

```
docker-compose.yml
├── postgres (PostgreSQL 16-alpine)    — persistent storage
├── n8n-import (one-shot)              — imports demo workflows on first run
└── n8n (n8nio/n8n:latest)             — workflow engine, port 5678
```

### Key decisions

| Decision | Rationale |
|----------|-----------|
| PostgreSQL only (no SQLite) | Reliable, production-like behavior without complexity |
| No Ollama/Qdrant | Simplified scope — use cloud AI providers (MiniMax) instead |
| `n8n:latest` tag | Always get newest features; pin a version if stability matters |
| OpenAI-compatible node for MiniMax | MiniMax exposes an OpenAI-compatible API; no custom node needed |
| No pre-imported credentials | Credentials require user's API key; must be created via UI |

## Stack

| Component | Image | Purpose |
|-----------|-------|---------|
| n8n | `n8nio/n8n:latest` | Workflow automation |
| PostgreSQL | `postgres:16-alpine` | n8n backend database |

## Environment variables

All configuration lives in `.env`. See `.env.example` for the template.

| Variable | Purpose |
|----------|---------|
| `POSTGRES_USER` | PostgreSQL username |
| `POSTGRES_PASSWORD` | PostgreSQL password (use a strong random value) |
| `POSTGRES_DB` | PostgreSQL database name |
| `N8N_ENCRYPTION_KEY` | Encrypts credentials stored in the database |
| `N8N_USER_MANAGEMENT_JWT_SECRET` | Signs authentication tokens |
| `MINIMAX_API_KEY` | API key for MiniMax (used in demo workflow) |

## File structure

```
.
├── docker-compose.yml          # Service definitions
├── .env.example                # Environment template
├── .env                        # Actual secrets (gitignored)
├── n8n/demo-data/
│   └── workflows/              # Auto-imported workflows
├── shared/                     # Mounted into n8n at /data/shared
├── specs/
│   ├── ARCHITECTURE.md         # Detailed architecture notes
│   └── TROUBLESHOOTING.md      # Common issues and solutions
├── AGENTS.md                   # This file — project governance
├── CLAUDE.md                   # Points to AGENTS.md
├── CONTRIBUTING.md             # Contribution guidelines
└── README.md                   # User-facing setup guide
```

## Conventions

### Docker Compose
- Use named volumes for persistence (`n8n_storage`, `postgres_storage`)
- Use a dedicated Docker network (`demo`)
- PostgreSQL must pass health checks before n8n starts
- The `n8n-import` service runs once to seed demo workflows

### Workflows
- Workflow JSON files go in `n8n/demo-data/workflows/`
- Credentials are NOT pre-imported — users create them via the n8n UI
- Use OpenAI-compatible nodes for third-party LLM providers

### Security
- Never commit `.env` (it's gitignored)
- Use strong random values for all secrets (32+ characters)
- `.env.example` uses placeholder values only

## What doesn't belong

- Production infrastructure (reverse proxies, SSL, monitoring, backups)
- Alternative databases or LLM backends
- Enterprise features (SSO, RBAC, multi-tenancy)
- Docker Compose profiles for GPU hardware

## Specs

Detailed documentation lives in `specs/`:
- [specs/ARCHITECTURE.md](specs/ARCHITECTURE.md) — how the services connect
- [specs/TROUBLESHOOTING.md](specs/TROUBLESHOOTING.md) — common issues and fixes
