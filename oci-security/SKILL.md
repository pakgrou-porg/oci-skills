---
author: Karl Miller
license: Apache 2.0
name: oci-security
description: "Design and implement OCI security: Cloud Guard, Security Zones, WAF, Vault/KMS, Bastion, encryption. Triggers: Cloud Guard, security zones, OCI Vault, KMS keys, secrets management, WAF, bastion, encryption at rest/in transit."
version: "1.0.0"
tags: ["oci", "security", "cloud-guard", "vault", "kms", "waf", "bastion", "encryption"]
---

# OCI Security - Cloud Guard, Security Zones, WAF, Vault/KMS, Bastion

## Overview
Implements threat detection, security posture enforcement, secrets management, WAF, and secure access. Mandatory dependency for all production workloads.

## 5-Step Process

### Step 1: Discovery
- Data classification levels; secrets inventory; WAF needs; Bastion access patterns; encryption requirements; compliance (PCI-DSS, HIPAA, SOC2)

### Step 2: Analysis
- Cloud Guard detector/responder recipes; Security Zone recipe restrictions; Vault topology; WAF rule sets; Bastion session types; key lifecycle

### Step 3: Creation

#### Cloud Guard: modules/security/cloud_guard.tf
```hcl
resource "oci_cloud_guard_cloud_guard_configuration" "config" {
  compartment_id   = var.tenancy_ocid
  reporting_region = var.home_region
  status           = "ENABLED"
}
resource "oci_cloud_guard_target" "tenancy_target" {
  compartment_id       = var.tenancy_ocid
  display_name         = "tenancy-target"
  target_resource_id   = var.tenancy_ocid
  target_resource_type = "COMPARTMENT"
}
```

#### Security Zones: modules/security/security_zones.tf
```hcl
resource "oci_cloud_guard_security_zone" "prod_zone" {
  compartment_id          = var.prod_compartment_id
  display_name            = "prod-security-zone"
  security_zone_recipe_id = oci_cloud_guard_security_recipe.prod_recipe.id
}
```

#### Vault/KMS: modules/security/vault.tf
```hcl
resource "oci_kms_vault" "app_vault" {
  compartment_id = var.security_compartment_id
  display_name   = "app-vault"
  vault_type     = "DEFAULT"
}
resource "oci_kms_key" "master_encryption_key" {
  compartment_id      = var.security_compartment_id
  display_name        = "master-encryption-key"
  management_endpoint = oci_kms_vault.app_vault.management_endpoint
  protection_mode     = "HSM"
  key_shape { algorithm = "AES"; length = 32 }
}
```

#### Secrets: modules/security/secrets.tf
```hcl
resource "oci_vault_secret" "db_password" {
  compartment_id = var.security_compartment_id
  vault_id       = oci_kms_vault.app_vault.id
  key_id         = oci_kms_key.master_encryption_key.id
  secret_name    = "db-admin-password"
  secret_content { content_type = "BASE64"; content = base64encode(var.db_admin_password) }
}
```

#### WAF: modules/security/waf.tf
```hcl
resource "oci_waf_web_app_firewall_policy" "app_waf_policy" {
  compartment_id = var.security_compartment_id
  display_name   = "app-waf-policy"
  request_protection {
    rules {
      name        = "owasp-protection"
      type        = "PROTECTION"
      action_name = "block-action"
      protection_capabilities {
        key     = "941100"
        version = 1
      }
    }
  }
}
```

#### Bastion: modules/security/bastion.tf
```hcl
resource "oci_bastion_bastion" "app_bastion" {
  compartment_id               = var.security_compartment_id
  bastion_type                 = "STANDARD"
  target_subnet_id             = var.private_subnet_id
  name                         = "app-bastion"
  client_cidr_block_allow_list = var.allowed_bastion_cidrs
  max_session_ttl_in_seconds   = 10800
}
```

#### OCI CLI Validation
```bash
oci cloud-guard configuration get --compartment-id $TENANCY_OCID
oci kms management vault list --compartment-id $SECURITY_COMPARTMENT_OCID
oci vault secret list --compartment-id $SECURITY_COMPARTMENT_OCID
oci waf web-app-firewall list --compartment-id $SECURITY_COMPARTMENT_OCID
oci bastion bastion list --compartment-id $SECURITY_COMPARTMENT_OCID
```

### Step 4: Validation
- Cloud Guard active with findings; Security Zone blocks non-compliant resources; Vault secret retrieval via resource principal; WAF blocks XSS/SQLi; Bastion reaches private hosts; HSM keys for prod

### Step 5: Handoff
- Vault/key/secret OCIDs -> all downstream skills
- WAF policy OCID -> oci-networking
- Bastion OCID -> oci-compute, oci-database
- Cloud Guard target -> oci-observability

## Security Baseline
- Cloud Guard ENABLED, all detector recipes active
- Security Zones on prod compartments
- Vault: HSM keys for prod, software for dev
- ALL secrets in Vault; zero in code/env
- WAF on all public LBs; OWASP Top 10
- Bastion: no direct SSH via public IP; 3hr TTL cap
- AES-256 at rest; TLS 1.2+ in transit
