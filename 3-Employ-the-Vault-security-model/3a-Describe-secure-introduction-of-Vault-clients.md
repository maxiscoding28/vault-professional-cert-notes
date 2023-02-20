## 3a. Describe secure introduction of Vault clients
**All conceptual. Describe with no labs**
### Goals
- Unique credentials for each application instance provisioned
- Limit exposure if credentials compromised
- Stop hardcoding credentials in codebase
- Reduce TTL of credentials as much as possible. No long-lived creds.
- Distribute credentials securely and only at runtime
- Use trusted platform to verify identities of clients
  - Use Approle if none avaiablable
- Employ orchestrator already authenticated to Vault to inject secrets
  - E.g Terraform to inject secrets into application build
