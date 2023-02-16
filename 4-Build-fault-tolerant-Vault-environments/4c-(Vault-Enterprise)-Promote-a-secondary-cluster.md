## 4c. (Vault Enterprise) Promote a secondary cluster
- DR Operation Token
  - Generated on DR cluster using unseal/recovery keys
  - Process is similar to generating a root
### On DR secondary cluster
```
vault operator generate-root -dr-token -init

vault operator generate-root -dr-token

vault operator generate-root -dr-token \
  -decode=$TOKEN \
  -otp=$OTP
```
### Demote primary
```
vault write -f sys/replication/dr/primary/demote
```
### Promote secondary
```
vault write sys/replication/dr/secondary/promote dr_operation_token=$DR_TOKEN
```
- DR Batch Token
  - Has a valid token ready in the event of a failure
  - Reduce time to generate DR operation token.
  - Need to ensure the TTL is valid