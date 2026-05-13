# Auth Methods

> **Why this matters:** Before Vault gives anyone a secret, it needs to know who is asking — auth methods are how it verifies identity, and choosing the right one matters for security.

---

## Before You Read This

This document assumes you understand Vault's path system and secrets engines.
If you haven't read it: [vault/01-secrets-engines.md](01-secrets-engines.md) — The plugins inside Vault that store and generate different types of secrets.

---

## What Authentication Means in Vault

**Authentication** — the process of proving who you are — is the first step in every Vault interaction. Before Vault returns a secret, it checks two things:

1. Who are you? (Auth Method)
2. Are you allowed to access this? (Policy — covered in [vault/03-policies.md](03-policies.md))

Vault does not have a single login system. It supports multiple **auth methods** — each designed for a different situation.

> Analogy: A building might accept a key card, a fingerprint, or a visitor badge to get through the front door. Each is a different auth method. Once inside, what rooms you can enter depends on your role — that's the policy.

### Check yourself
- [ ] I can explain what authentication means in the context of Vault
- [ ] I understand that Vault separates authentication (who you are) from authorization (what you can access)

---

## What a Token Is

After any successful authentication, Vault gives you a **token** — a temporary credential that represents your authenticated session.

Every subsequent request to Vault includes this token. Vault uses it to look up your policies and decide what you are allowed to do.

Tokens have a TTL (time-to-live). When the TTL expires, the token becomes invalid and you must re-authenticate.

```bash
# After authentication, set the token in your environment
export VAULT_TOKEN="hvs.XXXXXXXXXXXXXX"

# Now all vault commands use this token automatically
vault kv get secret/my-app/database
```

### Check yourself
- [ ] I can explain what a Vault token is
- [ ] I understand that tokens expire (TTL)
- [ ] I know how the token is used in subsequent requests

---

## Token Auth: The Simplest Method

**Token auth** is the most direct method: you authenticate by presenting a token directly. Vault ships with a root token when first initialized — this has full access and should only be used for initial setup.

```bash
vault login hvs.XXXXXXXXXXXXXX
```

Token auth is useful for:
- Initial setup and testing
- Machine-to-machine communication where you manage token distribution yourself

It is **not** suitable for most production use cases because you have to manage token rotation manually.

### Check yourself
- [ ] I know what the root token is and why it should be used carefully
- [ ] I understand the limitation of token auth for production

---

## AppRole: Auth for Applications and Services

**AppRole** is the auth method designed for automated systems — apps, scripts, CI/CD pipelines, and containers.

Instead of a username and password, AppRole uses two pieces:
- **Role ID** — a non-secret identifier (like a username) that identifies the app
- **Secret ID** — a short-lived, revocable credential (like a password) that proves the app is who it claims to be

```bash
# Enable AppRole
vault auth enable approle

# Create a role named "my-app"
vault write auth/approle/role/my-app \
  token_ttl=1h \
  token_max_ttl=4h

# Get the Role ID (share this with your app; it's not secret)
vault read auth/approle/role/my-app/role-id

# Generate a Secret ID (treat this like a password; it expires)
vault write -f auth/approle/role/my-app/secret-id

# Your app authenticates and receives a token
vault write auth/approle/login \
  role_id="<role-id>" \
  secret_id="<secret-id>"
```

> Analogy: Role ID is like your employee ID badge number — everyone can know it. Secret ID is like the PIN you enter with your badge — it's short-lived and you can revoke it immediately if it leaks.

### Check yourself
- [ ] I can explain the difference between Role ID and Secret ID
- [ ] I understand why AppRole is better suited for automated systems than token auth
- [ ] I know the commands to enable AppRole and create a role

---

## Other Auth Methods (Awareness Only)

You do not need to configure these now. Know they exist:

| Method | Used when |
|--------|----------|
| `userpass` | Human users logging in with a username + password |
| `github` | Developers authenticating with their GitHub account |
| `kubernetes` | Pods in a Kubernetes cluster authenticating automatically |
| `aws` | AWS services (EC2, Lambda) authenticating using their AWS identity |

As you grow, you will encounter these. The concept is always the same: prove identity, receive token, use token to access secrets within policy limits.

### Check yourself
- [ ] I can name two auth methods other than token and AppRole
- [ ] I understand that all auth methods end with the same result: a token

---

**What's next:** [vault/03-policies.md](03-policies.md) — How Vault decides what an authenticated identity is allowed to do.
