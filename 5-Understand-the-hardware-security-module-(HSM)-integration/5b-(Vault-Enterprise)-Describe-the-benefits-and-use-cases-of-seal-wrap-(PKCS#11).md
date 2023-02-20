## 5b. (Vault Enterprise) Describe the benefits and use cases of seal wrap (PKCS#11)
### What is seal wrapping?
- Vault already protect data using 256-bit AES.
- Extra layer of protection. Seal wrapping takes encrypted data and encrypts again using HSM keys
- Provides FIPS-140-2 compliance
- Allows Vault to be deployed in high security environment (PCI, HIPAA, DoD, NATO)
- Seal wrapped by default: recovery key, key shares, root key, keyring
- Also can seal wrap other values:
  - backend mounts (secret engines)
