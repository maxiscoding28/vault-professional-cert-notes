## 1g. Regenerate a root token
### About root tokens
- Unlimited access to vault with no TTL. Should not be used on a daily basis.

### How to regenerate a root token
- Requires quorum of recovery/unseal key holders
```
# Start generation
vault operator generate-root -init

# Input unseal/recovery
vault operator generate-root \
  -nonce=nonce
  $UNSEAL/RECOVERY_KEY

# Get token
vault operator generate-root 
  -decode=$ENCODED_TOKEN
  -otp=$OTP

# Get status of generation
vault operator generate-root -status

# Cancel generation
vault operator generate-root -cancel
```