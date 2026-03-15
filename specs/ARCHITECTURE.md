# Architecture

## Service topology

```
┌─────────────────────────────────────────────┐
│              Docker network: demo           │
│                                             │
│  ┌──────────┐         ┌──────────────────┐  │
│  │ postgres │◄────────│  n8n (port 5678) │  │
│  │ :5432    │         │                  │  │
│  └──────────┘         └──────────────────┘  │
│       ▲                       ▲              │
│       │                       │              │
│  ┌────┴─────────┐      ┌─────┴────┐        │
│  │ n8n-import   │      │ shared/  │        │
│  │ (one-shot)   │      │ volume   │        │
│  └──────────────┘      └──────────┘        │
└─────────────────────────────────────────────┘
```

## Startup sequence

1. **PostgreSQL** starts and runs health checks (`pg_isready`)
2. **n8n-import** waits for PostgreSQL to be healthy, then imports demo workflows from `n8n/demo-data/workflows/`
3. **n8n** waits for both PostgreSQL (healthy) and n8n-import (completed), then starts the web server on port 5678

## Data persistence

| Volume | Mount point | Contents |
|--------|------------|----------|
| `postgres_storage` | `/var/lib/postgresql/data` | All n8n data (workflows, credentials, executions) |
| `n8n_storage` | `/home/node/.n8n` | n8n internal config and encryption keys |
| `./shared` (bind mount) | `/data/shared` | User files accessible from n8n workflows |

## External connections

The demo workflow connects to:
- **MiniMax API** (`https://api.minimaxi.com/v1`) — OpenAI-compatible chat completions endpoint

n8n's 400+ integrations can connect to any external service. Credentials are encrypted in PostgreSQL using `N8N_ENCRYPTION_KEY`.

## Environment variable flow

```
.env file
  │
  ├──► PostgreSQL: POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB
  │
  └──► n8n: DB connection (via POSTGRES_*), N8N_ENCRYPTION_KEY, N8N_USER_MANAGEMENT_JWT_SECRET
```

`MINIMAX_API_KEY` is stored in `.env` for reference but must be manually entered as an n8n credential via the UI (n8n encrypts credentials separately).
