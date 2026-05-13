# Policies

> **Why this matters:** Knowing who you are is only half the picture — policies are how Vault decides what each identity is actually allowed to do, which paths they can read, and which they can never touch.

---

## Before You Read This

Policies work alongside auth methods. Make sure you understand:
- [vault/02-auth-methods.md](02-auth-methods.md) — How Vault verifies who you are before giving you access to anything.
- [vault/00-what-is-vault.md](00-what-is-vault.md) — especially the section on how Vault uses paths.

---

## What a Policy Is

A **policy** is a set of rules that says: "for these paths, these operations are allowed."

Every token in Vault has one or more policies attached to it. When a request comes in, Vault checks: does this token's policy allow this operation on this path? If yes, proceed. If no, deny.

By default, **everything is denied** unless a policy explicitly allows it. This is the principle of least privilege — start with nothing, grant only what is needed.

> Analogy: A policy is like an office building's access control list. The cleaning staff can access supply rooms and hallways but not the server room. Executives can access the server room but not HR filing cabinets. Access is specific and intentional.

### Check yourself
- [ ] I can explain what a Vault policy does in one sentence
- [ ] I understand the "deny by default" rule
- [ ] I understand what "least privilege" means

---

## What a Policy Looks Like

Policies are written in **HCL** (HashiCorp Configuration Language) — a simple, human-readable format.

Here is a policy that allows a `my-app` service to read its own secrets:

```hcl
# Allow my-app to read secrets at its own path
path "secret/data/my-app/*" {
  capabilities = ["read"]
}
```

Breaking it down:
- `path` — the Vault path this rule applies to. `*` is a wildcard (matches anything after that prefix).
- `capabilities` — the list of allowed operations

The available capabilities are:

| Capability | What it allows |
|------------|---------------|
| `read` | Read existing data |
| `list` | List keys at a path (but not their values) |
| `create` | Create new data |
| `update` | Modify existing data |
| `delete` | Delete data |
| `sudo` | Access root-protected paths (use sparingly) |

### Check yourself
- [ ] I can read a basic HCL policy and explain what it allows
- [ ] I can name at least four capabilities
- [ ] I understand what the `*` wildcard does in a path

---

## Writing and Applying a Policy

Save your policy to a file, then upload it to Vault:

```bash
# Save to a file
cat > my-app-policy.hcl << EOF
path "secret/data/my-app/*" {
  capabilities = ["read"]
}
EOF

# Upload to Vault
vault policy write my-app my-app-policy.hcl
```

Then attach the policy to an auth role so tokens get it automatically:

```bash
# Attach the policy to the AppRole we created in the auth methods doc
vault write auth/approle/role/my-app \
  token_policies="my-app"
```

Now every token issued to `my-app` via AppRole will have the `my-app` policy attached — and can only read from `secret/data/my-app/*`.

### Check yourself
- [ ] I know the command to upload a policy to Vault
- [ ] I know how to attach a policy to an AppRole
- [ ] I understand the connection between auth methods, tokens, and policies

---

## A Practical Example: Two Apps, Two Policies

Imagine two services: `frontend` and `backend`. Each should only access its own secrets.

```hcl
# frontend-policy.hcl
path "secret/data/frontend/*" {
  capabilities = ["read"]
}
```

```hcl
# backend-policy.hcl
path "secret/data/backend/*" {
  capabilities = ["read", "list"]
}

path "secret/data/shared/*" {
  capabilities = ["read"]
}
```

- `frontend` cannot read backend secrets
- `backend` can also read from a shared path
- Neither can write or delete

This is least privilege in practice: each service gets exactly what it needs, nothing more.

### Check yourself
- [ ] I can write a basic policy that grants read access to a specific path
- [ ] I can explain why two separate policies are better than one shared policy
- [ ] I understand the concept of least privilege in my own words

---

**What's next:** [vault/04-vault-and-docker.md](04-vault-and-docker.md) — How to run Vault inside Docker and get secrets into your containers.
