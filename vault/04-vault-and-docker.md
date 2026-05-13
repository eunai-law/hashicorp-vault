# Vault and Docker Together

> **Why this matters:** This is where both tools connect — running Vault in a Docker container and pulling secrets into your application containers without ever hardcoding anything.

---

## Before You Read This

This document requires a solid understanding of both tools. Make sure you have covered:
- [docker/05-docker-compose.md](../docker/05-docker-compose.md) — How to define and run multiple containers together as one system.
- [vault/03-policies.md](03-policies.md) — How Vault decides what an authenticated identity is allowed to do.

---

## Running Vault in Docker (Dev Mode)

The fastest way to get Vault running is in **dev mode** — a single-command, in-memory Vault server. It is not for production (data is lost on restart), but it is perfect for learning and development.

```bash
docker run -d \
  --name vault \
  --cap-add=IPC_LOCK \
  -p 8200:8200 \
  -e 'VAULT_DEV_ROOT_TOKEN_ID=dev-root-token' \
  -e 'VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200' \
  hashicorp/vault:latest
```

Breaking down the flags:
- `--cap-add=IPC_LOCK` — Vault needs this to prevent secrets from being swapped to disk
- `-p 8200:8200` — exposes Vault's API on your machine at port 8200
- `VAULT_DEV_ROOT_TOKEN_ID` — sets the root token to a predictable value for dev use
- `VAULT_DEV_LISTEN_ADDRESS` — tells Vault to listen on all interfaces inside the container

Test that it is running:

```bash
curl http://localhost:8200/v1/sys/health
```

### Check yourself
- [ ] I understand what "dev mode" is and why it is not for production
- [ ] I can explain what the `--cap-add=IPC_LOCK` flag does
- [ ] I know how to verify Vault is running after starting the container

---

## Connecting Your App Container to Vault

Your app container needs to reach Vault. The cleanest way is a shared Docker network.

```yaml
# docker-compose.yml
services:
  vault:
    image: hashicorp/vault:latest
    cap_add:
      - IPC_LOCK
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: "dev-root-token"
      VAULT_DEV_LISTEN_ADDRESS: "0.0.0.0:8200"

  my-app:
    image: my-app-image
    environment:
      VAULT_ADDR: "http://vault:8200"
      VAULT_TOKEN: "dev-root-token"
    depends_on:
      - vault
```

Because both services are on the same Compose network, `my-app` can reach Vault using the hostname `vault` — exactly the service name.

The `VAULT_ADDR` environment variable tells Vault client libraries where to find Vault. Any app using the official Vault SDK or CLI picks this up automatically.

> Analogy: The two containers are colleagues in the same office. Docker Compose put them on the same floor (network). They can walk to each other's desk (connect by hostname) without going through the building's public entrance.

### Check yourself
- [ ] I understand why both containers need to be on the same network
- [ ] I know what `VAULT_ADDR` and `VAULT_TOKEN` environment variables do
- [ ] I can explain why `vault` (the service name) works as a hostname inside Compose

---

## Putting Secrets Into Vault and Reading Them in Your App

**Step 1: Write a secret (from your terminal, not inside the container)**

```bash
export VAULT_ADDR="http://localhost:8200"
export VAULT_TOKEN="dev-root-token"

vault secrets enable -path=secret kv-v2
vault kv put secret/my-app/config db_password="my-real-password"
```

**Step 2: Read the secret from inside your app**

In Python (using the `hvac` library):

```python
import hvac
import os

client = hvac.Client(
    url=os.environ["VAULT_ADDR"],
    token=os.environ["VAULT_TOKEN"]
)

secret = client.secrets.kv.v2.read_secret_version(
    path="my-app/config",
    mount_point="secret"
)

db_password = secret["data"]["data"]["db_password"]
```

The app never has the password written in its code. It asks Vault at runtime.

### Check yourself
- [ ] I can walk through the steps of writing a secret and reading it from an app
- [ ] I understand why the password is never in the application code
- [ ] I know what the `hvac` library is used for

---

## What a Production Setup Looks Like (Awareness Only)

Dev mode is not for production. In production:

- Vault is **initialized and unsealed** (a multi-step process that protects the encryption keys)
- Vault uses **persistent storage** (a database or cloud storage backend, not memory)
- Applications authenticate using **AppRole** (from [vault/02-auth-methods.md](02-auth-methods.md)) — not the root token
- Vault is typically run as a cluster (multiple instances) for high availability

You do not need to configure this now. The patterns you have learned (secrets engines, auth methods, policies) all apply in production — only the infrastructure changes.

### Check yourself
- [ ] I understand why dev mode is not suitable for production
- [ ] I can name two differences between a dev and production Vault setup
- [ ] I know that AppRole (not the root token) is what apps use in production

---

**What's next:** [vault/CHECKLIST.md](CHECKLIST.md) — Master self-assessment for everything in the Vault section.
