## 6d. (Vault Enterprise) Create a paths filter
- For allowing or denying path to replicate to PR secondary clusters
- Feature for GDPR
- Can be set when enabling PR via UI or as a separate API call
  - https://developer.hashicorp.com/vault/api-docs/system/replication/replication-performance#create-paths-filter
```
vault write sys/replication/performance/primary/paths-filter/pr-secondary \
    mode="deny" paths="auth/USA_1/,auth/amurica2/"
```
- Can also mark an auth method or secret engine as local to not replicate
```
vault auth enable -path=gdpr_userpass -local userpass
```