## 4a. Configure a highly available (HA) cluster
- Do not create a single vault cluster - no redundancy, no failure tolerance, no scalability
### What should a cluster look like?
- Redundancy, failure tolerance, scalability
- For Vault Ent, limited to Integrated Storage or Consul storage backend
- Vault Operations will NOT feature Consul as a configuration or deployment option
### Multi-node cluster with integrated storage
- IS allows Vault nodes to provide its own replication storage across node withi na cluster
- Define a local path to store replicated data
  - `path` in storage stanza
  - `node_id` is unique id in cluster
- All data is replicated among all nodes in the cluster
- Can join nodes manually `vault operator raft join <ADDR>` or with `retry_join` in config file
  - define an IP or SAN plus port e.g. `http://127.0.0.1:8200`
  - `auto_join` with cloud tagging e.g. `provider=aws region=us-east-1 tag_key=foo tag_value=bar`
- Vault operator raft commnds
```
vault operator raft list-peers
vault operator raft join
vault operator raft remove-peer
vault operator raft snapshot
```
- Steps to join a node manually:
```
# Login to leader node
vault operator init
vault operator unseal

# Join seconds node
vault operator raft join $ADDR

# If using auto-unseal, steps are done
# If using shamir, need to unseal follower with leader's unseal key
```