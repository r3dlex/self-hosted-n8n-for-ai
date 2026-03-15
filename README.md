# Self-hosted n8n with PostgreSQL

A zero-install, Docker Compose based setup for running [n8n](https://n8n.io/) with PostgreSQL. Includes a demo chatbot workflow powered by [MiniMax](https://www.minimaxi.com/).

## What's included

- [**n8n**](https://n8n.io/) — Low-code workflow automation with 400+ integrations and AI nodes
- [**PostgreSQL 16**](https://www.postgresql.org/) — Reliable data storage for n8n

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)
- A [MiniMax API key](https://www.minimaxi.com/) (for the demo chatbot workflow)

## Quick start

```bash
git clone <this-repo>
cd self-hosted-n8n-for-ai
cp .env.example .env
```

Edit `.env` and set strong, unique values for:
- `POSTGRES_PASSWORD`
- `N8N_ENCRYPTION_KEY`
- `N8N_USER_MANAGEMENT_JWT_SECRET`
- `MINIMAX_API_KEY` (your MiniMax API key)

Then start:

```bash
docker compose up -d
```

Open <http://localhost:5678/> to set up your n8n account (first time only).

## Demo workflow — MiniMax Chatbot

The included workflow uses MiniMax as a chat model via the OpenAI-compatible API.

After first launch:

1. Open the workflow at <http://localhost:5678/workflow/srOnR8PAY3u4RSwb>
2. Click the **MiniMax Chat Model** node
3. Click **Create New Credential** and configure:
   - **API Key**: your MiniMax API key
   - **Base URL**: `https://api.minimaxi.com/v1`
4. Save the credential, then click **Chat** at the bottom to test

## Upgrading

```bash
docker compose pull
docker compose up -d
```

## Accessing local files

A `shared/` folder is mounted into the n8n container at `/data/shared`. Use this path in nodes that interact with the filesystem:

- [Read/Write Files from Disk](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.filesreadwrite/)
- [Local File Trigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.localfiletrigger/)

## Stopping

```bash
docker compose down       # stop services, keep data
docker compose down -v    # stop services and delete all data
```

## License

Apache License 2.0 — see [LICENSE](LICENSE).
