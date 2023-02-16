## 1h. Rekey Vault and rotate encryption keys
### What is a rekey?
- Creates a new set of Vault recovery/unseal keys and root key
- Allow to re-configure number of keys and threshold
- Required a threshold of key to rekey
- Provides a nonce value to given key holders

### Why would I need to rekey?
- Key is lost
- Employee leaves organization
- Organization requires a reky after period of time
- No downtime when performing this operation. Vault will continue to service requests.

### What is a key rotation?
- Rotates encryption key used to protect data on storage backend
- Encryption key never available to users. Does not require threshold of key/recovery key holders
- New keys added to keyring, old values can still decrypt
- Can be done online. No downtime
- Required policy:
```
vault policy write rotate-policy - <<EOF
path "sys/rotate" {
   capabilities = ["update", "sudo"]
}
path "sys/key-status" {
   capabilities = ["read"]
}
EOF
```