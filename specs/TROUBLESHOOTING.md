# Troubleshooting

Common issues and their solutions. This document grows as new problems are encountered.

---

## Docker & startup

### n8n won't start — "database does not exist"

**Symptom**: n8n logs show `error: database "n8n" does not exist`

**Fix**: Ensure `POSTGRES_DB` in `.env` matches what PostgreSQL was initialized with. If you changed `POSTGRES_DB` after first run, delete the volume and restart:

```bash
docker compose down -v
docker compose up -d
```

### Port 5678 already in use

**Symptom**: `Bind for 0.0.0.0:5678 failed: port is already allocated`

**Fix**: Stop the other process using port 5678, or change the port mapping in `docker-compose.yml`:

```yaml
ports:
  - 5679:5678  # use localhost:5679 instead
```

### n8n-import runs migrations but workflow doesn't appear

**Symptom**: Import logs show migrations completing and "Successfully imported 1 workflow" but the workflow isn't visible in n8n.

**Fix**: You need to create an owner account first. Visit <http://localhost:5678/> and complete the setup form. The workflow will then appear.

---

## Credentials & API keys

### "Authentication failed" when using MiniMax Chat Model

**Symptom**: Workflow execution fails with 401 or authentication error.

**Fix**:
1. Verify your MiniMax API key is valid at <https://www.minimaxi.com/>
2. In n8n, click the MiniMax Chat Model node → edit credential
3. Confirm the API key and base URL (`https://api.minimaxi.com/v1`) are correct

### Credentials lost after `docker compose down -v`

**Symptom**: Previously configured credentials are gone after restart.

**Cause**: The `-v` flag deletes all volumes, including the PostgreSQL database where credentials are stored.

**Fix**: Use `docker compose down` (without `-v`) to keep your data. Only use `-v` when you want a completely fresh start.

---

## Data & volumes

### Changes to .env don't take effect

**Symptom**: Updated passwords or keys in `.env` but services still use old values.

**Fix**: Restart the services:

```bash
docker compose down
docker compose up -d
```

Note: If you change `POSTGRES_PASSWORD` after the database was already initialized, you must also delete the postgres volume (`docker compose down -v`) since PostgreSQL only sets the password on first initialization.

### N8N_ENCRYPTION_KEY changed — can't decrypt credentials

**Symptom**: n8n starts but workflows fail with credential decryption errors.

**Cause**: `N8N_ENCRYPTION_KEY` was changed after credentials were already saved. Credentials are encrypted with the original key and cannot be decrypted with a new one.

**Fix**: Either restore the original `N8N_ENCRYPTION_KEY` or start fresh:

```bash
docker compose down -v
docker compose up -d
```

Then re-create your credentials in the n8n UI.
