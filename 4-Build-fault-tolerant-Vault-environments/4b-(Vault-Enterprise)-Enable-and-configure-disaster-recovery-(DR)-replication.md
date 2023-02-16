## 4b. (Vault Enterprise) Enable and configure disaster recovery (DR) replication
### What is Vault Replication?
- Need infrastructure to span multiple datacenters
- Vault needs to scale with common policies, consistent secrets and configurations regardless of datacenter
- Leader (Primary) - Follower (Secondary) Model. Primary is system of record and replicates async to secondary
- All communication between primary and secondaries is encrypted with mTLS
### Performance Replication
- Replicates underlying configuration, policies, and other data
- Service reads from client requests
- Clients authenticate to performance secondary separately
- Does not replicate tokens or leases to performance secondaries
### Disaster Replication
- Replication config, policies, and all other data
- Cannot service reads from client requests
- Will replicate tokens and leases created on the primary
### Networking Requirements
- Commiunication between clusters must be permitted - RPC forwarding and cluster bootstrapping
- If using DNS, each cluster must be able to resolve name to other cluser
- Port 8200 (Cluster Bootstrap), 8201 (RPC transfer data)
### Steps for setting up
- Activate DR on primary
- Create a secondary token on primary cluster
- Activate DR on secondary
```
vault write -f sys/replication/dr/primary/enable

vault write sys/replication/dr/primary/secondary-token id="dr-secondary"

vault write sys/replication/dr/secondary/enable token=$TOKEN

vault read sys/replication/dr/status
```