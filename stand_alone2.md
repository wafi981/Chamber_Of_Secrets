# 🔐 Vault Fundamentals – From Dev to Production: Stand Alone Part 2

This repository is a **hands-on learning course** for understanding **HashiCorp Vault** from first principles and gradually moving toward **real-world, production-grade usage**, including **Kubernetes** and **GitHub Actions CI/CD**.

This is not a copy‑paste guide.
This is a *concept‑first, production‑thinking* walkthrough inspired by how big companies manage secrets.

---

## 🎯 Course Goals

By the end of this course, you will:

* Understand **what Vault is and why it exists**
* Know the difference between **Dev mode vs Production mode**
* Understand **Seal / Unseal**, **Auth methods**, and **Policies** deeply
* Use **KV v1 vs KV v2** correctly
* Deploy Vault **locally, on Kubernetes (kind)**
* Store secrets safely (no env vars, no plaintext files)
* Prepare Vault for **GitHub Actions CI/CD using OIDC**

---

## 🧠 Mental Model (Read This First)

Vault is **NOT**:

* A `.env` file manager
* A Kubernetes Secret replacement (directly)
* A folder full of secret files

Vault **IS**:

* A secure API for secrets
* Identity‑aware
* Policy‑driven
* Encrypted at rest and in transit
* Designed for **dynamic, short‑lived access**

```
Client → Auth → Policy → Secrets Engine → Encrypted Storage
```

Everything you do in Vault maps to this flow.

---

## 🏗️ Vault Deployment Modes (Concept)

| Mode       | Purpose                             |
| ---------- | ----------------------------------- |
| Dev Mode   | Learning & testing only             |
| Standalone | Single-node production setup        |
| HA         | Enterprise-grade clusters           |
| External   | Injector-only (uses external Vault) |

This course intentionally starts in **Dev mode**, then moves to **Standalone on Kubernetes**.

---

## 📦 Environment Used

* macOS
* Vault CLI
* Kubernetes (kind)
* Vault Helm Chart
* GitHub Actions (later)

---

## 🚀 Part 1: Running Vault (Dev / Local)

### 1️⃣ Check Vault Status

```bash
vault status
```

This confirms:

* Vault is initialized
* Vault is unsealed
* Storage backend in use

Key fields to notice:

* `Seal Type`
* `Sealed: false`
* `Storage Type`

---

### 2️⃣ Login Using Root Token

```bash
vault login
```

The **root token** has unlimited power.

⚠️ In real production:

* Root token is stored offline
* Never used by CI/CD
* Used only for initial setup

---

## 🔌 Part 2: Secrets Engines

### What is a Secrets Engine?

A secrets engine is a **pluggable component** responsible for:

* Storing secrets
* Generating secrets
* Rotating secrets

Examples:

* KV (Key‑Value)
* Database
* AWS
* PKI

---

### 3️⃣ Enable KV v2 Secrets Engine

```bash
vault secrets enable -path=secret kv-v2
```

This **mounts** a KV v2 engine at the logical path `secret/`.

Important:

* This does NOT create files
* This does NOT touch the filesystem
* This creates a **logical API namespace**

---

## 📁 KV v1 vs KV v2 (Important)

| Feature     | KV v1 | KV v2 |
| ----------- | ----- | ----- |
| Versioning  | ❌ No  | ✅ Yes |
| Soft delete | ❌ No  | ✅ Yes |
| Rollback    | ❌ No  | ✅ Yes |
| Metadata    | ❌ No  | ✅ Yes |

KV v2 is **mandatory for production**.

---

## 📝 Part 3: Writing and Reading Secrets

### 4️⃣ Write a Secret

```bash
vault kv put secret/app/db username=admin password=supersecret
```

Vault response shows:

* Secret path
* Version number
* Creation timestamp

Internally, KV v2 stores this at:

```
secret/data/app/db
```

(The CLI abstracts this away.)

---

### 5️⃣ Read a Secret

```bash
vault kv get secret/app/db
```

Read a single field:

```bash
vault kv get -field=password secret/app/db
```

---

## 🗄️ How Vault Actually Stores Secrets (Critical Concept)

Vault **does not store secrets as files**.

Even though the pod contains:

```
/vault/data
```

This directory contains:

* Encrypted binary data
* Internal indexes
* Logical storage structures

Your secret paths like:

```
secret/app/db
```

Are **logical references**, not filesystem paths.

### Why this matters

* Stealing the disk does nothing
* `kubectl exec` cannot reveal secrets
* Access always requires authentication + policy checks

---

## 🔐 Encryption & Seal / Unseal

Vault encrypts everything using a **master key**.

* Master key is split using **Shamir Secret Sharing**
* Vault starts in a **sealed** state
* Unseal reconstructs the master key in memory

Without unseal → secrets are cryptographically inaccessible.

---

## ☸️ Part 4: Vault on Kubernetes

In this course we deploy Vault using:

* Helm chart
* Standalone mode
* File storage backend
* kind cluster (local)

This mirrors **real production topology** minus HA.

---

## 🔑 Part 5: Policies (Coming Next)

Policies answer one question:

> **Who can access what?**

Root policy is powerful but unsafe.

We will:

* Create least‑privilege policies
* Restrict paths like `secret/admin/*`
* Prepare Vault for CI/CD access

---

## 🔐 Part 6: GitHub Actions + OIDC (Coming Next)

We will integrate:

* GitHub OIDC JWT
* Vault JWT auth method
* Short‑lived tokens
* No static secrets in GitHub

This is **real enterprise CI/CD security**.

---

## 🧭 Learning Philosophy

This repository intentionally:

* Explains *why*, not just *how*
* Avoids insecure shortcuts
* Mirrors real-world practices

If you understand this repo fully, you **understand Vault**.

---

## 🚧 Roadmap

* [x] Vault fundamentals
* [x] KV v2 secrets
* [x] Kubernetes deployment
* [ ] Policies deep dive
* [ ] GitHub OIDC auth
* [ ] CI/CD secret injection
* [ ] Vault Agent & CSI driver

---

## 🧠 Final Note

Vault is not hard.

**Security without understanding is dangerous.**

This course fixes that.

