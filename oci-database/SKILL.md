---
author: Karl Miller
license: Apache 2.0
name: oci-database
description: "Design and implement OCI databases: Autonomous DB, MySQL HeatWave, NoSQL, DB Systems. Triggers: Autonomous Database, ATP, ADW, MySQL HeatWave, NoSQL, database provisioning, DB system."
version: "1.0.0"
tags: ["oci", "database", "autonomous-db", "mysql", "nosql", "atp"]
---

# OCI Database - Autonomous DB, MySQL HeatWave, NoSQL, DB Systems

## Overview
Database services: Autonomous (ATP/ADW), MySQL HeatWave, NoSQL, and traditional DB Systems. Covers provisioning, networking, encryption, backups, and connection patterns.

## 5-Step Process

### Step 1: Discovery
- Workload type (OLTP, OLAP, document, key-value); capacity; HA/DR requirements; connection patterns (private endpoint, mTLS); backup retention

### Step 2: Analysis
- Autonomous vs managed DB System; MySQL HeatWave for analytics; NoSQL for high-throughput key-value; OCPU/storage auto-scaling; private endpoint vs public

### Step 3: Creation

#### Terraform: modules/database/autonomous.tf
```hcl
resource "oci_database_autonomous_database" "app_db" {
  compartment_id           = var.compartment_id
  display_name             = "${var.project_name}-atp"
  db_name                  = var.db_name
  db_workload              = "OLTP"
  cpu_core_count           = var.cpu_cores
  data_storage_size_in_tbs = var.storage_tb
  is_auto_scaling_enabled  = true
  admin_password           = var.admin_password  # from Vault
  subnet_id                = var.db_subnet_id
  nsg_ids                  = var.db_nsg_ids
  private_endpoint_label   = "appdb"
  is_mtls_connection_required = true
  kms_key_id               = var.encryption_key_id
  vault_id                 = var.vault_id
}
```

#### Terraform: modules/database/mysql.tf
```hcl
resource "oci_mysql_mysql_db_system" "app_mysql" {
  compartment_id      = var.compartment_id
  display_name        = "${var.project_name}-mysql"
  shape_name          = var.mysql_shape
  subnet_id           = var.db_subnet_id
  availability_domain = var.availability_domain
  admin_username      = "admin"
  admin_password      = var.mysql_admin_password
  data_storage_size_in_gb = var.mysql_storage_gb
  is_highly_available = var.is_production
}
```

### Step 4: Validation
- Private endpoint only; mTLS enforced; encryption with CMK; backups configured; connection from app tier via NSG; admin password in Vault

### Step 5: Handoff
- Connection strings -> oci-containers-oke, oci-compute (as Vault secrets); DB OCID -> oci-observability, oci-disaster-recovery

## Security Baseline
- Private endpoints only; mTLS required; CMK encryption; admin passwords in Vault; NSG restricting to app-tier; automated backups; audit logging enabled
