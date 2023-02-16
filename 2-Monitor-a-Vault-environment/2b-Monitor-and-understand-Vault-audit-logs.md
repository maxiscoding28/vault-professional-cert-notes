## 2b. Monitor and understand Vault audit logs
 -  Detailed list of all authenticated requests and responses (in JSON)
 - Sensitive info is hashed with a salt. Never in plaintext
 - Audit types:
  - File
    - Write to a file
    - Does not assist with log rotation
    - Use fluentd or similar tool to send to a collector
  - Syslog
    - Write audit logs to a syslog
    - Send to local agent only
  - Socket
    - Write to TCP, UPD, or unix socket
    - TCP should be used to guarantee delivery of logs
  - Enable more than one audit device
    - If vault cannot write to a log it will stop responding to client requests
    - replicated vs. local indicates whether secondariesalso have audit device
  ```
  vault audit list

  vault audit enable -path=local_logs -local=true file file_path=/var/log/local_audit.log

  vault audit disable local_logs
  ```