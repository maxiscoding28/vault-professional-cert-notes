## 5a. (Vault Enterprise) Describe the benefits of auto unsealing with HSM
### What is a HSM?
- Network based physical device that can safeguard digital keys
- Keys can be used for encryption/decryption. Digital signatures, strong authentication and other functions.
- HSMs have tamper resistance. Detection of tampering can invoke response such as deleting keys
- Large eneterprises often have HSMs
- Public clouds offer access to dedicated or shared HSM
  - AWS CloudHSM or Azure Dedicate HSM is an HSM service
  - AWS KMS is example of a shared HSM service where multiple customers use service backed by HSM
### General HSM Support
  - Protect root key using HSM to encrypt/decrypt root key
  - Auto unseal Vault using wrapped key on local storage
  - Seal wrapping to provide extra layer of protection for FIPS 140-2 compliance
  - Entropy augumentation to generate randomness for cryptographic operations
  - Requires HSM that supports PKCS11 Standard
### Initializing Vault
  - Pass Root key from Vault to HSM
  - HSM encrypts root key
  - Returns encrypted root key
### Unsealing Vault
  - Retrieve encrypted root key from backend
  - Send encrypted root key to HSM to decrypt
  - HSM returns decrypted root key
  - Vault uses decrypted root key to decrypt encryption key
  - Encryption key stored in memory
