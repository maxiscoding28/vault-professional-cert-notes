## 1d. Implement integrated storage for open source and enterprise vault
### Cluster Management
- Manually join a node to a cluster
```
vault operator raft join $API_ADDR
```
- Automatically join using retry_join in the config file
```
    retry_join = {
        leader_api_addr = $LEADER_ADDR
    }
```
- Change leader node
```
vault operator step-down
```
### Snapshots
- Manually run snapshot
```
vault operator raft snapshot save test.snap
```
- Configurate automatic snapshots. Will immediately take snapshot on successful write. 
```
vault write sys/storage/raft/snapshot-auto/config/hourly \
  interval=1h \
  retain=24 \
  storage_type=local \
  path_prefix=/etc/vault.d/snapshots \
  local_max_space=20
```
- List or read snapshots config
```
vault list sys/storage/raft/snapshot-auto/config

vault read sys/storage/raft/snapshot-auto/config/hourly
```