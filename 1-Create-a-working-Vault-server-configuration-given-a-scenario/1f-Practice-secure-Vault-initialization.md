## 1f. Practice secure Vault initialization
### Generate PGP Key
```
gpg --gen-key
# Don't set a passphrase

gpg --export maxypoo | base64 > pgp.asc
```

### Initialize
- Create the root key and key/recovery shares
  - Options to define thresholds, shares
    - recovery_ for auto unseal, key_ for shamir
  - Key/recovery shares (distributed to team - Encrypts root key) => Root Key (Encrypts encryption key) => Encryption key (encrypts data)
- One operator runs init. How to prevent them from knowing all the keys?
  - encrypt with PGP, `-pgp-keys=<keys>` `-recovery-pgp-keys=<pub key file>`
```
vault operator init -recovery-shares=1 -recovery-threshold=1 -recovery-pgp-keys=pgp.asc
```
  - decrypt `echo $PGP_KEY | base64 --decode | gpg -dq
```

```
- Unseal
- Configure
  - Revoke intitial root token after initial configuration is complete.
