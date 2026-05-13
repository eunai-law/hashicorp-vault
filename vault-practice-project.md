# Vault Practice Project: Postgres Credentials via Vault

A dead-simple HTTP service that fetches Postgres credentials from Vault instead
of a `.env` file. Built with Python, FastAPI, and Docker Compose.

---

## What You Will Practice

| Vault Feature | What You Do |
|---|---|
| **KV secrets engine** | Store your Postgres password in Vault instead of a `.env` file |
| **Dynamic database secrets** | Let Vault generate short-lived Postgres users on demand |
| **Token auth** | Your app authenticates to Vault with a token to get secrets |
| **Policies** | Limit your app so it can only read the one secret it needs |

---

## Architecture

```
docker-compose.yml
├── vault     (dev mode — no unsealing needed)
├── postgres  (you already know this one)
└── app       (FastAPI, one endpoint: GET /db-status)
```

Flow:
1. App starts → authenticates to Vault with a token
2. App fetches Postgres credentials from Vault
3. App connects to Postgres using those credentials
4. `GET /db-status` returns whether the connection succeeded

---

## Project Structure

```
vault-practice/
├── docker-compose.yml
├── app/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
└── vault/
    └── init.sh       # seeds Vault with the Postgres secret on first boot
```

---

## docker-compose.yml

```yaml
services:
  vault:
    image: hashicorp/vault:latest
    container_name: vault
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: "dev-root-token"   # never use in prod
      VAULT_DEV_LISTEN_ADDRESS: "0.0.0.0:8200"
    cap_add:
      - IPC_LOCK
    networks:
      - vaultnet

  postgres:
    image: postgres:16
    container_name: postgres
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: supersecret
      POSTGRES_DB: appdb
    networks:
      - vaultnet

  app:
    build: ./app
    container_name: app
    ports:
      - "8000:8000"
    environment:
      VAULT_ADDR: "http://vault:8200"
      VAULT_TOKEN: "dev-root-token"
    depends_on:
      - vault
      - postgres
    networks:
      - vaultnet

networks:
  vaultnet:
```

---

## vault/init.sh

Run this once after `docker compose up` to seed Vault with the Postgres secret.

```bash
#!/bin/sh
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=dev-root-token

# Enable the KV v2 secrets engine
vault secrets enable -path=secret kv-v2

# Store the Postgres credentials
vault kv put secret/app/database \
  host="postgres" \
  port="5432" \
  dbname="appdb" \
  username="admin" \
  password="supersecret"

echo "Vault seeded."
```

Run it with:
```bash
docker exec vault sh < vault/init.sh
```

---

## app/requirements.txt

```
fastapi
uvicorn
hvac        # official Vault Python client
psycopg2-binary
```

---

## app/Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY main.py .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## app/main.py

```python
import hvac
import psycopg2
from fastapi import FastAPI
import os

app = FastAPI()

def get_db_creds():
    client = hvac.Client(
        url=os.environ["VAULT_ADDR"],
        token=os.environ["VAULT_TOKEN"],
    )
    secret = client.secrets.kv.v2.read_secret_version(
        path="app/database",
        mount_point="secret",
    )
    return secret["data"]["data"]


@app.get("/db-status")
def db_status():
    creds = get_db_creds()
    try:
        conn = psycopg2.connect(
            host=creds["host"],
            port=creds["port"],
            dbname=creds["dbname"],
            user=creds["username"],
            password=creds["password"],
        )
        conn.close()
        return {"status": "ok", "source": "vault"}
    except Exception as e:
        return {"status": "error", "detail": str(e)}
```

---

## Steps to Run

```bash
# 1. Start everything
docker compose up -d

# 2. Seed Vault with the Postgres secret (once)
docker exec vault sh < vault/init.sh

# 3. Hit the endpoint
curl http://localhost:8000/db-status
# → {"status":"ok","source":"vault"}
```

---

## Stretch Goals (once the basics work)

### 1. Dynamic Database Secrets
Instead of storing a static password in Vault, let Vault create a temporary
Postgres user with an expiry time. Vault handles rotation automatically.

```bash
# Enable the database secrets engine
vault secrets enable database

# Configure it to talk to your Postgres container
vault write database/config/appdb \
  plugin_name=postgresql-database-plugin \
  allowed_roles="app-role" \
  connection_url="postgresql://{{username}}:{{password}}@postgres:5432/appdb" \
  username="admin" \
  password="supersecret"

# Create a role that generates short-lived users (1 hour TTL)
vault write database/roles/app-role \
  db_name=appdb \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Test it — Vault generates a fresh user each time
vault read database/creds/app-role
```

Then update `main.py` to call `vault.secrets.database.generate_credentials("app-role")`
instead of the KV path. Every app restart gets a fresh, expiring Postgres user.

### 2. Write a Policy
Lock your app down so its token can only access `secret/app/database` and nothing else.

```bash
# Write a policy file
vault policy write app-policy - <<EOF
path "secret/data/app/database" {
  capabilities = ["read"]
}
EOF

# Create a token scoped to that policy
vault token create -policy="app-policy"
# Use this token in your app instead of the root token
```

### 3. Secret Rotation
Update the password stored in Vault and confirm your app picks it up on the
next request without a restart:

```bash
vault kv put secret/app/database \
  host="postgres" port="5432" dbname="appdb" \
  username="admin" password="newpassword123"
```

---

## Why This Matters for Your Discord Bot

Your bot currently stores Postgres and Redis credentials as env vars or in a
`.env` file. With Vault you could:

- Store all credentials in Vault and have the bot fetch them at startup
- Use dynamic secrets so a compromised bot token cannot expose your DB password
  permanently — Vault-issued credentials expire automatically
- Rotate secrets without restarting the bot (just re-fetch from Vault)
