## 6a. Use batch tokens
- Encrypted binary blogs. Not persisted to storage. Lightweight. 
- Ideal for high volume operations. Lots of reads or transits.
- If orphan, can be used across all clusters within the the replica set.
- Differences from service token:
  - Can't be root 
  - Can't create child tokens
  - Can't be renewed. No max_ttl
  - Can't be listed.
  - Can't be revoked.
  - Can't be periodic
  - Has no accessors
  - Has no cubbyhole
  - Stops working if parent revoked (if not orphan)
  - Can be used across PR clusters
- Can be used for DR opertation (promote secondary)
```
vault token create -orphan=true -type=batch
```