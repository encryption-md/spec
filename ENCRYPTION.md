# ENCRYPTION

> Encryption implementation standards for AI agent projects.
> Spec version: 1.0 | Full specification: https://encryption.md

---

## PURPOSE
# Define the technical encryption standards agents must enforce.
# Companion to ENCRYPT.md (data classification) — this file defines
# HOW to protect data; ENCRYPT.md defines WHAT to protect.

---

## REQUIREMENTS
# Algorithm and key requirements. All implementations must meet these.

algorithms:
  symmetric:
    approved:
      - AES-256-GCM               # Authenticated encryption — preferred
      - ChaCha20-Poly1305         # Alternative for resource-constrained environments
    forbidden:
      - DES                       # Deprecated — do not use
      - 3DES                      # Deprecated — do not use
      - AES-128-ECB               # ECB mode forbidden — not semantically secure
      - RC4                       # Broken — do not use

  asymmetric:
    approved:
      - RSA-4096                  # Minimum 4096-bit for new implementations
      - ECDSA P-384               # NIST P-384 or higher
      - Ed25519                   # Preferred for signatures in new systems
    forbidden:
      - RSA-1024                  # Insufficient key length
      - RSA-2048                  # Acceptable legacy only — do not use for new
      - DSA                       # Deprecated

  hashing:
    approved:
      - SHA-256                   # Minimum for general use
      - SHA-384                   # Required for TLS 1.3 cipher suites
      - SHA-512                   # Preferred for high-assurance contexts
      - BLAKE3                    # Approved for non-cryptographic hashing
    forbidden:
      - MD5                       # Broken — do not use
      - SHA-1                     # Deprecated — do not use

key_lengths:
  minimum_symmetric_bits: 256
  minimum_asymmetric_bits: 4096
  minimum_ecc_bits: 384

key_rotation:
  data_encryption_keys:
    max_age_days: 90              # Rotate every 90 days
    on_breach: immediate          # Rotate immediately on suspected compromise
  tls_certificates:
    max_age_days: 365             # Annual rotation minimum
    preferred_age_days: 90        # Quarterly preferred
  api_keys:
    max_age_days: 30              # Monthly rotation
    on_breach: immediate          # Revoke and replace on any suspected compromise

---

## IMPLEMENTATION
# Specific implementation requirements for TLS and key management.

tls:
  minimum_version: "1.3"         # TLS 1.3 required for all new connections
  forbidden_versions:
    - "1.0"
    - "1.1"
    - "1.2"                      # Discouraged — only if legacy forces it
  cipher_suites:
    required:
      - TLS_AES_256_GCM_SHA384
      - TLS_CHACHA20_POLY1305_SHA256
    forbidden:
      - RC4-SHA
      - DES-CBC3-SHA
      - NULL-SHA
  certificate_validation: strict  # Never skip certificate validation
  hsts_enabled: true              # HTTP Strict Transport Security required
  hsts_max_age_seconds: 31536000  # 1 year minimum

key_storage:
  approved_sources:
    - environment_variables       # Injected at runtime from secure store
    - secrets_manager             # AWS Secrets Manager, HashiCorp Vault, etc.
    - hardware_security_module    # HSM for highest assurance requirements
  forbidden:
    - hardcoded_in_source         # Never hardcode keys in code
    - plaintext_config_files      # Never store in .env committed to VCS
    - agent_memory_long_term      # Never persist keys in agent memory

certificate_management:
  auto_renewal: true              # Automate certificate renewal
  renewal_buffer_days: 30         # Renew 30 days before expiry
  ca_approved:
    - Let's Encrypt               # Free, automated, widely trusted
    - DigiCert                    # Enterprise use
    - AWS Certificate Manager     # Cloud-native deployments

---

## COMPLIANCE
# Standards mapping for regulated environments.

standards:
  FIPS_140_3:
    applicable: true
    requirement: FIPS validated cryptographic modules where mandated
    scope: federal_and_regulated_sectors

  SOC2:
    applicable: true
    controls:
      - CC6.1: Logical and physical access controls
      - CC6.7: Transmission encryption required
      - CC7.2: Anomaly detection for security events

  ISO_27001:
    applicable: true
    controls:
      - A.10.1.1: Policy on use of cryptographic controls
      - A.10.1.2: Key management lifecycle
      - A.18.1.5: Regulation of cryptographic controls

  GDPR:
    applicable: true
    requirements:
      - Article_25: Data protection by design — encryption at rest
      - Article_32: Appropriate technical measures — AES-256 minimum
      - Article_33: Encrypted breach reduces notification scope

---

## AUDIT

log_file: .encryption.log
log_format: jsonl
log_fields:
  - timestamp
  - event_type              # key_rotation, cert_renewal, violation_detected
  - algorithm_used
  - key_id
  - compliance_check_passed
  - session_id

---

## METADATA

owner: your-name-or-org
contact: ops@example.com
last_reviewed: 2026-03-11
review_frequency: quarterly
spec_version: "1.0"
spec_url: https://encryption.md
