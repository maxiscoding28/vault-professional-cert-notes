## 1c. Auto unseal Vault
### KMS unseal
_**May be asked to migrate from shamir to KMS in lab environment**_
- Vault nodes must be able to access KMS key for cloud provider. For AWS, encrypt, decrypt, describeKey
- Example vault config:
```
seal "awskms" {
  region     = "us-west-2"
  kms_key_id = "arn:aws:kms:us-east-1:XXX
}
```
- Migrate with `vault operator unseal -migrate` (use for each unseal key)
- After migrate, shamir keys become recovery keys
### Transit Unseal
- On unseal node, Enable transit, create key, create unseal policy, and generate periodic token
```
vault secrets enable transit

vault write -f transit/keys/autounseal

vault policy write unseal-policy - <<EOF
path "transit/encrypt/autounseal" {
  capabilities = ["update"]
}
path "transit/decrypt/autounseal" {
  capabilities = ["update"]
}
path "transit/keys" {
  capabilities = ["list"]
}
path "transit/keys/autounseal" {
  capabilities = ["read"]
}
EOF

vault token create -orphan -policy="unseal-policy" -period=24h
```
- Add transit seal stanza config for new node
```
seal "transit" {
  address = "http://unseal-cluster:8200"
  disable_renewal = "false"
  key_name = "autounseal"
  mount_path = "transit/"
  tls_skip_verify = "true"
  
  # Add Vault token to config or set as VAULT_TOKEN env when starting transit
  token = ""
}
```