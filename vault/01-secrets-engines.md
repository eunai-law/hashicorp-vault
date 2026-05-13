# Secrets Engines

> **Why this matters:** Vault can store many different kinds of secrets — secrets engines are the modular plugins that handle each type, so you only enable what you actually need.

---

## Before You Read This

This document assumes you understand what Vault is and how it uses paths.
If you haven't read it: [vault/00-what-is-vault.md](00-what-is-vault.md) — The secrets problem Vault solves and what Vault is at its core.

---

## What a Secrets Engine Is

A **secrets engine** — the plugin that stores or generates a specific type of secret — is the core building block of Vault's storage system.

Different secrets engines handle different use cases:

| Engine | What it stores or does |
|--------|----------------------|
| `kv` | Key-value pairs (the simplest: store any secret by name) |
| `database` | Generates temporary, single-use database credentials |
| `pki` | Issues TLS certificates |
| `aws` | Generates temporary AWS access credentials |
| `ssh` | Issues signed SSH certificates |

You enable only the engines you need. Each engine is mounted at a path.

> Analogy: Think of Vault as a Swiss Army knife. Each secrets engine is one of the tools — a blade, scissors, a screwdriver. You flip out the tools you need and leave the rest folded in.

### Check yourself
- [ ] I can explain what a secrets engine is in my own words
- [ ] I can name at least three types of secrets engines
- [ ] I understand that secrets engines are mounted at paths

---

## The KV Secrets Engine (Start Here)

The **KV (Key-Value) engine** is the simplest secrets engine. It works exactly like a dictionary: you store a secret under a key, and you retrieve it by that key.

There are two versions:
- **KV v1** — basic storage, no history
- **KV v2** — stores a version history of every secret (you can roll back to a previous value)

Enable the KV v2 engine at the path `secret/`:

```bash
vault secrets enable -path=secret kv-v2
```

Write a secret:

```bash
vault kv put secret/my-app/database password="super-secret-123"
```

Read it back:

```bash
vault kv get secret/my-app/database
```

The secret is now stored at the path `secret/my-app/database` under the key `password`.

### Check yourself
- [ ] I understand the difference between KV v1 and KV v2
- [ ] I know the command to enable a secrets engine
- [ ] I know the commands to write and read a KV secret
- [ ] I can explain what "mounted at a path" means for a secrets engine

---

## Dynamic Secrets: A Step Further

Most secrets engines you will learn about later (database, AWS, PKI) generate **dynamic secrets** — credentials that are created on demand and expire automatically.

This is more secure than static secrets because:
- Every app gets its own unique credential
- Credentials expire after a set time (time-to-live, or TTL)
- If a credential leaks, it becomes useless after expiration
- Vault logs exactly who requested what credential and when

You do not need to configure dynamic secrets now. Just know the concept exists and why it matters.

> Analogy: A static secret is like one master key that everyone copies. A dynamic secret is like a one-time keycard that self-destructs after 24 hours. Much harder to abuse.

### Check yourself
- [ ] I can explain what a dynamic secret is
- [ ] I can name at least two reasons dynamic secrets are more secure than static ones
- [ ] I know what TTL (time-to-live) means in this context

---

**What's next:** [vault/02-auth-methods.md](02-auth-methods.md) — How Vault verifies who you are before giving you access to anything.
