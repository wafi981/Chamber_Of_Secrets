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


## Initialise Yoru Vault Server:

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
 
