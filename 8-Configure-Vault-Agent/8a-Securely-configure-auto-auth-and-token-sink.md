## 8a. Securely configure auto-auth and token sink
### What is the Vault Agent?
- Vault agent is client daemon that runs alongside an application.
- Enables legacy application to interact and consume secrets
- Vault features include:
  - auto authentication including renewal
  - Secure/delivery storage of tokens (response wrapping)
  - Local caching of secrets
  - Templating
### Vault Agent Auto-Auth
- The Vault agent uses a pre-defined auth method to authenticate to Vault and obtain token
- The token is stored in the "sink" which is a flat file on a local file system.
- The application can read this token and invoke Vault API directly
- This strategy allows Vault Agent to manager the token guarantee a valid token is always available to the application
- Auth methods allowed by agent
  - AliCloud
  - AppRole
  - AWS
  - Azure
  - Ceritificate
  - CloudFoundary
  - GCP
  - JWT
  - Kerberos
  - K8s
- Example configuration file
```
auto_auth {
  method "approle" {
    config = {
      role_id_file_path   = "/vault/agent/role-id"
      secret_id_file_path = "/vault/agent/secret-id"
      remove_secret_id_file_after_reading = false
    }
  }

  sink "file" {
    config = {
      path = "/vault/token"
    }
  }
}
```
### Vault Agent Sink
- File is only supported method of storing the auto-auth token
- Common config parameters include:
  - type = what type of sink to use (only file is available)
  - path = location of the file
  - mode = change file permissions (default is 640)
  - wrap_ttl = retrieve the token using response wrapping
### Prtoect token with response wrapping
- Response wrapping at auth method
  - Authenticate to vault. Token is wrapped by Vault server
  - Server returns wrapped token over network. Protects against MITM attacks.
  - Downside: Agent can't renew the token. Wrap token only has use limit of 1
- Response wrapping at sink
  - Agent can renew the token
  - Does not protect against MITM attacks (token is sent over network)
  - Application has wrapped token in the sink but vault agent can update/renew when needed
- Wrap_ttl in auto_auth = wrapped by auth method
- Wrap_ttl in sink file = wrapped by sink
