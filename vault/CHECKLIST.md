# Vault: Master Self-Assessment Checklist

Check these off as you **understand** them — not just as you read them.
Re-reading before checking a box is encouraged. That is the point.

---

## 00 — What Is HashiCorp Vault
[Full document](00-what-is-vault.md)

- [ ] I can name at least three types of secrets an application might need
- [ ] I can explain at least two problems with hardcoding secrets in code
- [ ] I understand what "rotating a secret" means
- [ ] I can explain what Vault does in one sentence
- [ ] I can name the four core concepts of Vault (secrets engines, auth methods, policies, tokens)
- [ ] I understand how Vault organizes everything using paths

**I understand the problem Vault solves and what it is at a high level.**

---

## 01 — Secrets Engines
[Full document](01-secrets-engines.md)

- [ ] I can explain what a secrets engine is in my own words
- [ ] I can name at least three types of secrets engines
- [ ] I understand that secrets engines are mounted at paths
- [ ] I understand the difference between KV v1 and KV v2
- [ ] I know the command to enable a secrets engine
- [ ] I know the commands to write and read a KV secret
- [ ] I can explain what a dynamic secret is
- [ ] I know what TTL (time-to-live) means for a dynamic secret

**I can store and retrieve secrets using Vault's KV secrets engine.**

---

## 02 — Auth Methods
[Full document](02-auth-methods.md)

- [ ] I can explain what authentication means in Vault
- [ ] I understand that Vault separates authentication from authorization
- [ ] I can explain what a Vault token is
- [ ] I understand that tokens have a TTL
- [ ] I know what the root token is and why it should be used carefully
- [ ] I can explain the difference between Role ID and Secret ID in AppRole
- [ ] I understand why AppRole is better suited for automated systems
- [ ] I know the commands to enable AppRole and create a role
- [ ] I can name two auth methods other than token and AppRole
- [ ] I understand that all auth methods end with the same result: a token

**I understand how Vault verifies identity and issues tokens.**

---

## 03 — Policies
[Full document](03-policies.md)

- [ ] I can explain what a Vault policy does in one sentence
- [ ] I understand the "deny by default" rule
- [ ] I understand the principle of least privilege
- [ ] I can read a basic HCL policy and explain what it allows
- [ ] I can name at least four policy capabilities
- [ ] I understand what the `*` wildcard does in a Vault path
- [ ] I know the command to upload a policy to Vault
- [ ] I know how to attach a policy to an AppRole
- [ ] I can write a basic policy that grants read access to a specific path
- [ ] I can explain why separate policies per service are better than one shared policy

**I can write, upload, and apply Vault policies.**

---

## 04 — Vault and Docker Together
[Full document](04-vault-and-docker.md)

- [ ] I understand what Vault dev mode is and why it is not for production
- [ ] I can explain what `--cap-add=IPC_LOCK` does
- [ ] I know how to verify Vault is running after starting the container
- [ ] I understand why both containers need to be on the same Docker network
- [ ] I know what `VAULT_ADDR` and `VAULT_TOKEN` environment variables do
- [ ] I can walk through writing a secret and reading it from an app
- [ ] I understand why the password is never in the application code
- [ ] I can name two differences between a dev and production Vault setup
- [ ] I know that AppRole is what apps use to authenticate in production

**I can run Vault in Docker and connect an application to it.**

---

## All Done?

If every box above is checked, you have a solid working understanding of HashiCorp Vault.

Return to: [README.md](../README.md) — full learning path overview and Mermaid diagrams showing how both tools connect.
