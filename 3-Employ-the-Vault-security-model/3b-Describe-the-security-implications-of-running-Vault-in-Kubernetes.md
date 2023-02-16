## 3b. Describe the security implication of running Vault in Kubernetes
### Running Vault on K8s
- Deploy with Helm chart
- TLS End to End Encryption
  - Dont offload TLS at load balancer
    - Ensures end to end encryption from client to the Vault nodes
  - Use TLS certificates signed bya trusted CA
  - Requires TLS 1.2+
- Disable Core Dumps
  - Core dump files may include Vault encryption key
  - Ensure RLIMIT_CORE is 0 or use ulimit -c 0 command inside container
- Ensure mlock is enabled
  - Ensures memory from process isn't swapped to disc
  - IPC_LOCK, prevents encryption keys from swapping to disk
- Container Supervisor
  - Don't run as root. Process might escape container also have root in node
  - SecurityContext, PodSecuritContext runAsNonRoot
  - Dont run Vault as Root