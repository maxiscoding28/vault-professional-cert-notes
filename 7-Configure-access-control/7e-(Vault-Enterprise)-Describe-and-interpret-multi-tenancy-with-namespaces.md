## 7e. (Vault Enterprise) Describe and interpret multi-tenancy with namespaces
- Provide isolated environments on a single Vault environment
- Multi-tenant but centralized management
- Allows delegation of Vault responsibilities
- Namespaces have their own:
  - Policies
  - Auth methods
  - Secret Engines
  - Tokens
  - Identities
- Do not have their own
  - Storage backend
  - Audit devices
- Set vault namespace in request
```
export VAULT_NAMESPACE=$NAMESPACE

# or

vault kv get -namespace=$NAMESPACE kv/data/sql/prod

# or

curl \
  -H "X-Vault-Token: $TOKEN" \
  -request GET \
  https:/vault.com:8200/v1/$NAMESPACE/kv/data/sql/prod

# or

curl \
  -H "X-Vault-Token: $TOKEN" \
  -H "X-Vault-Namespace: $NAMESPACE"
  -request GET \
  https:/vault.com:8200/v1/kv/data/sql/prod
```
- Create namespace
```
vault namespace create $NAMESPACE
```