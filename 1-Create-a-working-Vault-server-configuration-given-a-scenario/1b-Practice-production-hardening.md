## 1b. Practice production hardening
**Expected to discuss these concepts but not expected to configure (in a lab enviroment)**

**May be asked how to improve security based on a provided configuration**
### General Recommendations
- Deployment model
  - Fewer shared resources the better. Single tenancy. Focus on protecting memory contents
  - Most to Least isolated environment: Hardaware > VMs > Containers.
  - Many customer use virtualization or containerization but deploy to dedicated clusters
- Limit access to Vault nodes
  - Prevent SSH/RDP, AWS SSM, `kube exec` or `docker exec`
  - Instead, access Vault via API or CLI from workstation or jump box
  - If you really need direct access use Hashicorp Boundary
- Limit services running on Vault Nodes
  - Vault nodes should be dedicated to Vault services
  - You should not have other services contending for resources. 
    -  More services == more firewall requirements
  - Encryption keys are stored in memory
  - Exception: telemetry agent, log file agent
- Permit only required ports on firewall
  - Allow inbound ports: Default ports for Vault: 8200, 8201
  - Allow outbound ports e.g. DB 3306, snapshots 443, ldap 389
  - Don't allow UI, SSH ports
- Immutable upgrades
  - Bring new nodes online and destroy old nodes
  - Take care to ensure replication complete for newly added nodes before removing old nodes.
- For integrated storage, hardware should have high IOPs. Data is stored local disc
- For Consul, data is stored in memory (on Consul nodes)
#### Operating System Recommendations
- Never run Vault as root. Can expose sensitive data
- Limit access to configuration files and folders to Vault user
- Vault will need access to write local files: database, audit logs, snapshots
- Secure Files and Directories
  - Protect snapshots, binaries, config files, plugin files, service config, audit device files etc.
- Protect storage backend
  - If using Consul, limit access to Consul Node, use ACLS, verify_server_hostname in config file
- Disable shell history
  - Possible to discover credentials/tokens in history
- Configure SELinux/AppArmor (if possible)
- Turn off core dumps
  - Core dumps can reveal encryption keys (stored in memory)
- Protect and audit the vault service file
  - Know if file is modified or replaced. Attacker could point to compromised binaries to leak data
- Patch OS frequently
- Disable swap
  - Vault stores sensitive data in-memory, unencrypted. Data should never be written disk.
  - Enable mlock (for Linux)
#### Vault Specific Recommendations
- Secure vault with TLS
  - Ensure tls_disable does not equal true or 1 in config
  - Load balancers can terminate TLS or use pass through to vault nodes
- Enable Auditing
  - Use multiple audit devices
  - Send audit logs to a collection server
  - Archive log data and create alerts for certain actions
  - Possible alerts: use of root, new root, new auth, modify auth, audit log failures
- No cleartext credentials
  - Don't put credentials in configuration file. Use ENV vars
  - Use cloud integrated services (e.g. AWS IAM)
- Upgrade vault frequently
- Get rid of initial root token after initial setup
- Disable UI if not in use
- Secure unseal/recovery keys
  - Use PGP
  - Distribute to multiple team mebers
  - Dont store keys in vault
- Minimize TTLS for leases and tokens
  - Use smallest TTLS for token, define Max TTLS to prevent renewals.
  - Reduces burden on storage backend
- Least Privilege
  - Only access for required business function
  - Separate policies for apps and users
  - Limit use of * and + paths
  - Use templated policies
- Regular backups
  - Backup config files and directories
  - Automate vault backup using snapshots and regularly test
- Integrate with existing IdP
  - Using locally defined credentials is an administrative burden