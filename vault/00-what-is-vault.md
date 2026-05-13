# What Is HashiCorp Vault?

> **Why this matters:** Every application needs passwords, API keys, and certificates — Vault is the secure, centralized place to store and distribute them, so they never end up hardcoded in your code.

---

## The Secrets Problem

A "secret" is anything your app needs to keep private: a database password, an API key, a TLS certificate, an encryption key.

The bad (but common) way to handle secrets: paste them directly into your code or config files.

```python
# This is what we are trying to avoid
DB_PASSWORD = "super-secret-password-123"
API_KEY = "sk-abc123"
```

The problems with this:
- The secret lives in your git history forever
- Anyone with access to the code can read it
- Rotating (changing) a secret means editing files and redeploying
- You have no record of who accessed the secret, or when

**Vault** solves all of these problems at once.

> Analogy: Hardcoding secrets is like writing your ATM PIN on a sticky note attached to your card. Vault is a bank vault with a keypad — only the right people get in, and every entry is logged.

### Check yourself
- [ ] I can name at least three types of secrets an application might need
- [ ] I can explain at least two problems with hardcoding secrets in code
- [ ] I understand what "rotating a secret" means

---

## What Vault Is

**HashiCorp Vault** is a tool for storing, accessing, and controlling secrets.

Instead of your app reading a password from a config file, it asks Vault: "I am service X, I need the database password." Vault checks if service X is allowed to have it, and if so, returns it. Every request is logged.

Vault has four core concepts — each covered in its own document:

| Concept | What it does |
|---------|-------------|
| **Secrets Engines** | The plugins that store or generate different types of secrets |
| **Auth Methods** | How Vault verifies who (or what) is asking |
| **Policies** | The rules that say who can access what |
| **Tokens** | The credential Vault gives you after you authenticate |

> Analogy: Vault is a physical safe at a bank. The lock mechanism is Auth Methods. The bank's access rules are Policies. The safe's compartments are Secrets Engines. Your key card after signing in is a Token.

### Check yourself
- [ ] I can explain what Vault does in one sentence
- [ ] I can name the four core concepts of Vault
- [ ] I understand why Vault is better than storing secrets in environment variables or config files

---

## How Vault Is Organized Internally

Vault stores and retrieves everything through **paths** — similar to file paths on a computer.

```
secret/my-app/database-password
auth/token/lookup
sys/health
```

Every operation in Vault targets a path. When you read a secret, write a secret, or configure an auth method, you are interacting with a path. This is important because Policies (covered in [vault/03-policies.md](03-policies.md)) control access by path.

### Check yourself
- [ ] I understand that Vault organizes everything using paths
- [ ] I can give an example of what a Vault path might look like
- [ ] I understand why paths matter for access control

---

**What's next:** [vault/01-secrets-engines.md](01-secrets-engines.md) — The plugins inside Vault that store and generate different types of secrets.
