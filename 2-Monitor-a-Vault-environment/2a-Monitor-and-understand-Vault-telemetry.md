## 2a. Monitor and understand Vault telemetry
**Monitor and Understand == likely expected to discuss these concepts but not expected to configure (in a lab enviroment)**
**Possibly configure audit logs, create telemetry stanza (with provided parameters)**
  - Providers: statsite, stasd, circonus, dogstatsd, prometheus, stackdriver
  - e.g vault.core.handle_request, mem.used_percent
  - Configured in the configuration file `telemetry_stanza`. 
    - Need to bounce service or re-read config file to pick up changes.
  - May install telemetry agent on same node as vault