## 2c. Monitor and understand Vault operational logs
### Overview
- Valuable for troubleshooting.
- Levels (least to most detail): Error, Warn, Info, Debug, Trace
- Provides info when vault starts
#### Specify log level
- Use the CLI flag `-log-level`
- Set env variable `VAULT_LOG_LEVEL`
  - Take effect when vault is restarted
- Set `log_level` configuration parameter
  - Take effect when vault is restarted
- On modern linux or docker:
```
journalctl -b --no-pager -u vault

docker logs vault0
```