## 1a. Enable and configure (generic) secrets engines
### KV Secrets Engine
- Enabling version 1 or version 2 of KV secrets engine
```
vault secrets enable -path=kv1 kv

vault secrets enable -version=2 -path=kv2 kv
```
- Read mount configuration and tune configuration
```
vault read sys/mounts/kv1/tune

vault secrets tune -description="hello there" kv1
```
- Differences between v1 and v2 urls
```
# KV version=1
http://127.0.0.1:8200/v1/kv/max

# KV version=2
http://127.0.0.1:8200/v1/kv2/data/max
```
- KV get and put (uses different url paths under the hood for CLI)
```
vault kv get kv1/max
vault kv get kv2/max

vault kv put kv1/max pass=1234 username=max
vault kv put kv2/max pass=1234 username=max
```
- KV patch (v2 only). 
  - KV1 overwrites all data on update. 
  - KV2 can patch and retain other keys
```
vault kv put kv1/max pass=4321

vault kv patch kv2/max pass=4321
```
- KV2 versioned/metadata commands
  - Delete a version, undelete a version, rollback to a verion, delete all versions
```
vault kv delete -versions=1 kv2/max
vault kv undelete -versions=1 kv2/max
vault kv rollback -version=1 kv2/max
vault kv metadata delete kv2/max
```
- Upgrade KVv1 to v2 
```
vault kv enable-versioning path
```
---
### Database
- Enable Database secrets engine and configure a DB Connection (Command will fail if Vault can't connect to DB)
```
vault secrets enable database

vault write database/config/mysql \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(192.168.211.5:3306)/" \
  allowed_roles="cat-person" username=root password=root
```
- Create Role
```
vault write database/roles/cat-person \
  db_name=mysql \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON animals.cats TO '{{name}}'@'%';" \
  default_ttl="1h"
```
- Rotate root credentials for the config. Only Vault will know credentials after this is run.
```
vault write -f database/rotate-root/mysql
```
- Generate Credentials with role
```
vault read database/creds/cat-person
```
---
#### PKI
- Enable PKI secrets engine, generate root certificate and inspect it.
```
vault secrets enable pki

vault write -f pki/root/generate/exported common_name=max-root -format=json | jq -r '.data.certificate + "\n" + .data.private_key' > root.pem

openssl x509 -in root.pem -text -noout
```
- Generate int csr and inspect it
```
vault secrets enable -path=pki_int pki

vault write -format=json pki_int/intermediate/generate/internal \
     common_name="max-int" \
     | jq -r '.data.csr' > int.csr

openssl req -in int.csr -text -noout
```
- Sign int with root, inspect signed certificate and set-signed for pki_int mount
```
vault write -format=json pki/root/sign-intermediate \
    csr=@int.csr \
    | jq -r '.data.certificate' > int.cert.pem

openssl x509 -in int.cert.pem -text -noout

vault write pki_int/intermediate/set-signed certificate=@int.cert.pem
```
- Create role, issue, cert, inspect cert, revoke cert, tidy cert store
```
vault write pki_int/roles/max-role \
     allowed_domains="max.com" \
     allow_subdomains=true \
     ttl=12h

vault write -format=json pki_int/issue/max-role common_name=test.max.com \
    | jq -r '.data.certificate + "\n" + .data.private_key' > leaf.pem

openssl x509 -in leaf.pem -text -noout

vault write -f pki_int/revoke serial_number=$CERT_SERIAL_NUMBER

vault write -f pki_int/tidy tidy_cert_store=true
```
---
### Transit
- Create key, update config, rotate key
```
vault write -f transit/keys/max

vault write transit/keys/max/config min_encryption_version=2

vault write -f transit/keys/max/rotate
```
- Encrypt decrypt
```
vault write transit/encrypt/max plaintext=$B64_PLAINTEXT

vault write transit/decrypt/max ciphertext=$CIPHERTEXT

# Will fail due to min_encryption_version set to 2
vault write transit/encrypt/max key_version=1 plaintext=$B64_PLAINTEXT
```
---
### Identity
- Create policies and enable auth methods
```
~> vault policy list                               
default
entity-policy
external-group-policy
internal-group-policy
upass1-alias-policy
upass2-alias-policy
root

vault auth enable -path=upass1 userpass
vault auth enable -path=upass2 userpass
```
#### Entities
- Create user for each auth method, create entity, and assign respective policies
```
vault write auth/upass1/users/max1 password=1234 token_policies=upass1-alias-policy
vault write auth/upass2/users/max2 password=1234 token_policies=upass2-alias-policy

vault write identity/entity name=max-winslow policies=entity-policy
```
- Create entiy aliases using mount_accessor and entity id (canonical id)
```
vault write identity/entity-alias name=max1 canonical_id=$ENTITY_ID mount_accessor=$MOUNT_ACCESSOR1
vault write identity/entity-alias name=max2 canonical_id=$ENTITY_ID mount_accessor=$MOUNT_ACCESSOR2
```
- Login, `upass(1/2)-alias-policy` policies should be different but both logins should have `entity policy`
```
vault login -path=upass1 -method=userpass -no-store=true username=max1 password=1234
vault login -path=upass2 -method=userpass -no-store=true username=max2 password=1234
```
#### Groups
> By default, Vault creates an internal group. When you create an internal group, you specify the group members rather than group alias. Group aliases are mapping between Vault and external identity providers (e.g. LDAP, GitHub, etc.). Therefore you define group aliases only when you create external groups. For internal groups, you specify member_entity_ids and/or member_group_ids.

> External groups can have one (and only one) alias.

_Internal Group_
- Create internal-group using entity id (canonical_id)
```
vault write identity/group name="internal-group" policies=internal-group-policy member_entity_ids=$ENTITY_ID
```
- Login again, confirm internal-group-policy is associated with token
```
vault login -path=upass1 -method=userpass -no-store=true username=max1 password=1234
vault login -path=upass2 -method=userpass -no-store=true username=max2 password=1234
```
_External Groups_
- Enable Github auth, configure, create external group
```
vault auth enable github
vault write auth/github/config organization=$GITHUB_ORGANIZATION_NAME
vault write identity/group name=external-group policies=external-group-policy type=external
```
- Create group alias (only one group-alias for an external group)
```
vault write identity/group-alias name=$GITHUB_TEAM_NAME mount_accessor=$MOUNT_ACCESSOR canonical_id=GROUP_ID
```
- Login (need a Github token), confirm that external group policy is associated with token
```
vault login -method=github
```
---
### Cubbyhole
- Create default token and login
```
vault token create -policy=default
vault login $TOKEN
```
- Write and read back from cubbyhole (Only this token can access path)
```
vault write cubbyhole/max pass=1234
vault read cubbyhole/max
```
- Login as root, get wrap token for kv path and lookup token
  - Token only has 1 use and response-wrapping policy
```
vault login $ROOT_TOKEN

vault kv put kv/max pass=1234

vault kv get -wrap-ttl=60m kv/max
```
- Login as default, unwrap token to get value
```
vault token create -policy=default

vault login $DEFAULT_TOKEN

vault unwrap $WRAPPING_TOKEN
