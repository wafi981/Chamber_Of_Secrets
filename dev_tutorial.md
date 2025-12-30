# HashiCorp Vault – Dev Mode Hands-on Tutorial (Reference README)\

This document is a step-by-step learning reference for Vault Dev Mode, based exactly on the commands you ran. The goal is to build strong fundamentals so future concepts (GitHub Auth, OIDC, CI/CD, production Vault) make sense.

What does Vault do?
Well this is not your average secret storage system, vault can do a bit more like:

- Store secrets encrypted at rest
- Control who can access which secret (policies)
- Issue short-lived credentials (dynamic secrets)
- Authenticate machines & pipelines using identity (JWT, OIDC, cloud metadata)


## Installing The Vault

Using Brew (for mac)

```
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```

For other Operating system Refer: [Here](https://developer.hashicorp.com/vault/install)

Check if Installation was successful or not:

 ```
 vault --version
 ```

**NOTE: THIS IS A TUTORIAL FOR DEV MODE, DO NOT USE IT IN PROUCTION.**


## Initialise Your Vault Server:

```
 vault server -dev -dev-root-token-id root -dev-tls
 ```

In this command the first section along with the dev flag will be enough to initialise the vault sever, however I have added the last 2 flags for better understanding.

Normally in dev mode:
Vault generates a random root token and prints it to the console
With the ```-dev-root-token-id``` flag:
The root token is exactly root, useful for scripts, demos, or quick testing and of course it is Less secure (but acceptable in dev mode)

```-dev-tls```: This Enables HTTPS (TLS) for the dev server.
By default, Vault runs over plain HTTP, No certificates, No TLS handshake errors which is easier for quick local testing but again we add this just for our learning


## Set the required Environment Variables

Vault CLI talks to Vault server using env vars.
You have to set the following variables:

```
export VAULT_ADDR='https://127.0.0.1:8200'
export VAULT_CACERT='/var/folders/.../vault-ca.pem'
```

 These are your vault servers address and trust vaults TLS certificate, without these cli cannot securely connect

 ## Check Vault Status

 ```
 ~/Downloads % vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.21.1
Build Date      2025-11-18T13:04:32Z
Storage Type    inmem
Cluster Name    vault-cluster-4d8cce37
Cluster ID      aadaa8b0-2973-db2e-1adf-24a7c34e39af
HA Enabled      false

 ```

 Here is the value of sealed field is true means you will not be able to use it.
 
 High availability is not enabled in this mode as you can see from the last line


 ## Login to vault

 ```
 ~/Downloads % vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                root
token_accessor       LWYjBtZs9CmpD4aq5TCqZqT7
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

We configured this this token earlier in our commands. CLI auto-authenticates future commands.

⚠️ Root tokens are never used in real systems




## What are tokens?

What a Vault Token Is

A Vault token is the proof that a client (human or machine) is authenticated and authorized to perform actions in Vault.

No token = no access.


#### Every Vault request:
Must include a token, Is evaluated against policies attached to that token


Think of a token as: A short‑lived identity + permission bundle



#### How Tokens Are Issued

- Client authenticates using an auth method (userpass, GitHub, Kubernetes, OIDC, etc.)

- Vault verifies identity with a trusted source

- Vault issues a token

- Client uses the token for all future requests

Auth method ≠ Token :Auth method proves who you are

Token is what you use



#### Root Token (Special Case)

- Created at Vault initialization

- Full access (like root in Linux)

- No TTL (never expires)



#### ⚠️ Production rule:

Root tokens are ONLY for initial setup or emergencies



Token Metadata
Every token has:

```
Field	    Meaning
TTL	      How long token is valid
Max TTL	  Absolute lifetime
Renewable	Can be extended
Policies	 What it can do
Accessor	 Safe handle to manage token
```



#### Token accessors:
Can revoke / renew tokens

Cannot access secrets



### Token Types (Very Important)



#### 1. Service Tokens (Default)
Stored in Vault, Renewable, Revocable, Support cubbyhole


Used for:
- Humans

- CI/CD

- Long‑running services



Prefix: hvs.*


#### 2. Batch Tokens

Not stored, Not renewable, Lightweight, High scale

Used for:

- Massive container fleets

- Very short‑lived jobs

- Prefix: hvb.*


#### 3. Periodic Tokens

Have TTL, No max TTL, Must be renewed continuously

Used for:

- Long‑running services that cannot re‑authenticate


#### 4. Orphan Tokens

Not tied to parent token

Parent revocation does NOT revoke them

Used when:

- You want isolation from parent lifecycle



#### Token Lifecycle Summary

Create → Use → Renew → Revoke / Expire

Good security = short TTL + renew when needed
 
