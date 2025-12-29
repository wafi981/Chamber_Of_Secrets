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



 
