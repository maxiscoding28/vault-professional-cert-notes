## 6b. (Vault Enterprise) Describe the use cases of performance standby nodes
### OSS
- Standby does not provide read or write functionality
- If standby receives request, it forwards request to active (or redirects)
### Enterprise
- Standby node can response to read requests
- Write requests get forwarded to the active node
- Called "performance" standby.
### Eventual Consistency
- Active node writes data
- Data propagates out to standby
- Read after write can return old data or not return data
- Enabled by default, can be disabled with `disable_performance_standby=true` in config.
### Target performance standby
- `/sys/health` endpoint
- Load balancers can target specific return codes to determine active vs performance standby
- Default codes:
  - 200 initialized, unsealed, active
  - 429, unsealed but standby
  - 472, - DR secondary active
  - 473, perf standby
  - 501, not initialized
  - 503, sealed