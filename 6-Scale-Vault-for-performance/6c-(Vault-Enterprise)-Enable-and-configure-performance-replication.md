## 6c. (Vault Enterprise) Enable and configure performance replication
### Intro to Performance Replication
- Replicates config, policies and other data
- Can serve read requests
- Does not replicate tokens or leases to secondaries
- Reads and some writes (dyanmic creds) generated to PR. All other writes sent to primary.
- On primary
```
# Enable PR on primary
vault write -f sys/replication/performance/primary/enable

# Generate secondary token
vault write -f sys/replication/performance/primary/secondary-token \
  > id=pr-secondary
```
- On secondary
```
vault write sys/replication/performance/secondary/enable token=$TOKEN

vault read sys/replication/performance/status -format=json
```