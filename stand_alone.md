# HashiCorp Vault on kind (Standalone Mode)

This README documents the **end‑to‑end process of deploying HashiCorp Vault in Standalone mode on a local kind Kubernetes cluster**, explains **what is happening at each step**, and clarifies **Vault concepts like init, seal, and unseal**.

This setup mirrors **real production workflows** (used by large companies) while remaining safe for **local learning and CI/CD experimentation**.

---

## Architecture Overview

* **Cluster**: kind (local Kubernetes)
* **Vault Mode**: Standalone (single node)
* **Storage Backend**: File (PersistentVolume)
* **Seal Type**: Shamir
* **HA**: Disabled (single pod)
* **Use Case**: Learning, GitHub Actions integration, Kubernetes secrets

> ⚠️ Not recommended for real production, but **perfect for production‑style learning**.

---

## Step 1: Add HashiCorp Helm Repository

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
```

**Why?**
Helm charts are the official and safest way to deploy Vault in Kubernetes.

---

## Step 2: Install Vault (Standalone Mode)

```bash
helm install vault hashicorp/vault
```

### What this does

* Deploys **Vault Server** as a StatefulSet (`vault-0`)
* Creates a **PersistentVolumeClaim** for Vault data
* Deploys **Vault Agent Injector** (used later for auto‑injecting secrets into pods)

Default Helm behavior = **Standalone mode** (exactly what you wanted).

---

## Step 3: Verify Pods

```bash
kubectl get pods -A
```

Expected output:

* `vault-0` → Vault server
* `vault-agent-injector-xxxxx` → Injector webhook

Initially you saw:

```
READY: 0/1
STATUS: Running
```

This is **expected behavior**.

---

## Step 4: Why Vault Was Not Ready

From readiness probe:

```
Initialized: false
Sealed: true
```

### Explanation

Vault **starts sealed by design**.

* It **will not serve traffic** until:

  1. Initialized
  2. Unsealed

This is a **core security feature** — even if someone steals the disk, data remains encrypted.

---

## Step 5: Initialize Vault

```bash
kubectl exec -ti vault-0 -- vault operator init
```

### Output (example)

```
Unseal Key 1: key1
Unseal Key 2: key2
Unseal Key 3: key3
Unseal Key 4: key4
Unseal Key 5: key5

Initial Root Token: root
```

### What this means

* Vault generated **5 unseal key shares**
* **Any 3 of them** are required to unseal Vault
* Root token is created (full admin access)

### Production note

In real companies:

* Keys are distributed to **multiple people or systems**
* Root token is **never used daily**

---

## Step 6: Unseal Vault

Vault requires **3 out of 5 keys**.

```bash
kubectl exec -ti vault-0 -- vault operator unseal key1
kubectl exec -ti vault-0 -- vault operator unseal key2
kubectl exec -ti vault-0 -- vault operator unseal key3
```

### Final status

```
Sealed: false
Initialized: true
```

🎉 **Vault is now unsealed and operational**.

---

## Step 7: Verify Vault Is Ready

```bash
kubectl get pods
```

Expected:

```
vault-0   1/1   Running
```

Readiness probe passes because Vault can now answer API requests.

---

## Important Concepts Explained

### Seal / Unseal

* **Sealed** → Data encrypted, API locked
* **Unsealed** → Vault decrypts data into memory
* Happens on every restart

---

### Shamir Secret Sharing

Vault splits the master key into pieces:

* Total shares: 5
* Threshold: 3

This prevents a single person/system from controlling Vault.

---

### Storage = File

* Data stored on Kubernetes PersistentVolume
* Encrypted at rest
* Simple and reliable for standalone mode

---

### Why Standalone Mode Was the Right Choice

| Mode       | Why / Why Not                        |
| ---------- | ------------------------------------ |
| Dev        | ❌ In‑memory, auto‑unsealed, insecure |
| Standalone | ✅ Best for learning & CI/CD          |
| HA         | ❌ Overkill for local kind            |
| External   | ❌ Needs existing Vault               |

You made the **correct choice**.

---

## What You Achieved

✅ Production‑style Vault deployment
✅ Kubernetes‑native storage
✅ Manual init & unseal (real security flow)
✅ Vault Agent Injector ready

This is **exactly how real platforms start Vault**.

---

## Next Logical Steps

1. Login using root token
2. Enable KV secrets engine
3. Create policies
4. Enable Kubernetes auth
5. Connect GitHub Actions using JWT/OIDC
6. Inject secrets into pods
