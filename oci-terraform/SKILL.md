---
author: Karl Miller
license: Apache 2.0
name: oci-terraform
description: "Design and implement Terraform modules and OCI Resource Manager stacks for infrastructure as code. Triggers: Terraform, Resource Manager, IaC, state management, module design, OCI provider."
version: "1.0.0"
tags: ["oci", "terraform", "resource-manager", "iac", "state-management"]
---

# OCI Terraform - Modules, Resource Manager, State Management

## Overview
Infrastructure as Code with Terraform OCI provider and Resource Manager stacks. Covers module structure, state management, provider configuration, and RM stack deployment.

## 5-Step Process

### Step 1: Discovery
- Existing IaC state; module organization preferences; state backend (OCI Object Storage, Terraform Cloud); environment promotion strategy; RM stack vs CLI workflow

### Step 2: Analysis
- Module decomposition (per-skill or per-environment); variable/output design; state locking; provider version pinning; RM stack variables schema

### Step 3: Creation

#### Provider Configuration: providers.tf
```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    oci = { source = "oracle/oci"; version = "~> 5.0" }
  }
  backend "s3" {
    bucket                      = "terraform-state"
    key                         = "app/terraform.tfstate"
    region                      = "us-ashburn-1"
    endpoint                    = "https://<namespace>.compat.objectstorage.us-ashburn-1.oraclecloud.com"
    skip_region_validation      = true
    skip_credentials_validation = true
    skip_metadata_api_check     = true
    force_path_style            = true
  }
}
provider "oci" {
  tenancy_ocid = var.tenancy_ocid
  region       = var.region
}
```

#### Module Structure
```
terraform/
  modules/
    compartments/   (from oci-compartments)
    iam/            (from oci-iam)
    networking/     (from oci-networking)
    security/       (from oci-security)
    compute/        (from oci-compute)
    oke/            (from oci-containers-oke)
    functions/      (from oci-functions)
    storage/        (from oci-storage)
    database/       (from oci-database)
    observability/  (from oci-observability)
    devops/         (from oci-devops)
  environments/
    dev/    (main.tf, variables.tf, terraform.tfvars)
    staging/
    prod/
  stacks/           (Resource Manager zip packages)
```

#### Resource Manager Stack: stacks/schema.yaml
```yaml
title: "Application Infrastructure Stack"
schemaVersion: 1.1.0
version: "1.0.0"
variableGroups:
  - title: "General"
    variables: [tenancy_ocid, compartment_ocid, region]
  - title: "Networking"
    variables: [vcn_cidr, create_public_subnet]
  - title: "Compute"
    variables: [instance_shape, node_count]
variables:
  tenancy_ocid:
    type: string
    required: true
    title: "Tenancy OCID"
  region:
    type: oci:identity:region:name
    required: true
    title: "Region"
```

### Step 4: Validation
- terraform validate passes; terraform plan shows expected changes; state backend accessible with locking; RM stack deploys successfully; no drift between plan and apply

### Step 5: Handoff
- Module repository for team consumption; RM stack zip for Console deployment; state backend documentation

## Security Baseline
- State encrypted at rest in OCI Object Storage with CMK; state backend access restricted by IAM; no secrets in tfvars (use Vault data sources); provider auth via instance principal in automation; RM stack input variables never contain plaintext secrets
