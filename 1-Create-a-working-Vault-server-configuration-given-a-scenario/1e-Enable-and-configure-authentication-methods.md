## 1e. Enable and configure (generic) auth methods
### AppRole
- Enable approle, create role and secret, login
```
vault write auth/approle/role/max \
  token_policies=admin \
  token_ttl=1h \
  token_max_ttl=24h

vault read auth/approle/role/max/role-id

vault write -f auth/approle/role/max/secret-id

vault write auth/approle/login role_id=$ROLE_ID secret_id=$SECRET_ID
```
### Tokens
- Service tokens
  - Default token type. Start with `hvs`
  - Persist to storage, heavy storage reads/write
  - Can be renewew, revoked, create child tokens

- Batch tokens
  - Start with `hvb`. Designed to be lightweight, scalable
  - Not persisted to storage but not fully featured
  - Ideal for high volume ops such as encryption (transit)
  - Can be used for DR replication cluster promotion
  - Created with `vault token create -type=batch`
  - Can be used across PR cluster if orphan

- Periodidc tokens
  - Service token with no max_ttl. Can constantly renew if renewed before end of period
  - Created with `vault token create -period=24`
- Use limited token
  - Created with `vault token create -use-limit=2`
- Orphan token
  - Not affected by TTL/revocation of its parent token
  - Create with `vault token create -orphan`
-  Methods to authenticate with token:
```
$VAULT_TOKEN

-- header X-Vault-Token: $VAULT_TOKEN

--header "Authorization: Bearer $VAULT_TOKEN

vault login $VAULT_TOKEN
```
- Revoke token: `vault token revoke $VAULT_TOKEN`
   - For non-orphans, if token used to create token (parent) is revoked then so is child.
### UserPass
- Enable mount, create user, read user info, login as user
```
vault auth enable userpass

vault write auth/userpass/users/max password=1234

vault read auth/userpass/users/max

vault auth/userpass/login username=max password=1234

vault write auth/userpass/login/max password=1234
```