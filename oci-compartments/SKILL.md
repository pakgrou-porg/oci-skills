---
author: Karl Miller
license: Apache 2.0
name: oci-compartments
description: "Design and implement OCI compartment hierarchies, tagging strategies, budgets, quotas, and cost tracking. Triggers: compartment design, tenancy structure, OCI tagging, budgets, cost allocation, quota management."
version: "1.0.0"
tags: ["oci", "compartments", "tagging", "budgets", "governance", "cost-management"]
---

# OCI Compartments - Hierarchy, Tagging, Budgets, Quotas

## Overview
Designs compartment hierarchies enforcing resource isolation, cost attribution, and governance. Covers tagging, budget alerts, quota policies, cost tracking.

## 5-Step Process

### Step 1: Discovery
- Identify organizational units, compliance boundaries
- Map existing compartment structure
- Identify cost allocation requirements
- Determine environment tiers (dev/staging/prod/shared-services)

### Step 2: Analysis
- OCI compartment nesting: 6 levels max
- Service limits per compartment
- Cross-compartment access patterns
- Tag namespace constraints
- Budget threshold requirements

### Step 3: Creation

#### Reference Hierarchy
```
root (tenancy)
  +-- shared-services (networking, security, observability, identity)
  +-- applications
  |     +-- app-name-1 (dev, staging, prod)
  |     +-- app-name-2 (dev, staging, prod)
  +-- data (databases, storage, analytics)
  +-- platform (devops, ai-ml, integration)
```

#### Terraform: modules/compartments/main.tf
```hcl
variable "tenancy_ocid" { type = string }
variable "compartment_structure" {
  type = map(object({
    description = string
    children = optional(map(object({
      description = string
      children = optional(map(object({ description = string })), {})
    })), {})
  }))
}

resource "oci_identity_compartment" "level1" {
  for_each       = var.compartment_structure
  compartment_id = var.tenancy_ocid
  name           = each.key
  description    = each.value.description
  enable_delete  = false
  freeform_tags  = { "managed-by" = "terraform" }
}
```

#### Terraform: modules/compartments/tags.tf
```hcl
resource "oci_identity_tag_namespace" "project" {
  compartment_id = var.tenancy_ocid
  name           = "project-tags"
  description    = "Project-level resource tags"
}

resource "oci_identity_tag" "environment" {
  tag_namespace_id = oci_identity_tag_namespace.project.id
  name             = "environment"
  is_cost_tracking = true
  validator { validator_type = "ENUM"; values = ["dev", "staging", "prod", "shared"] }
}

resource "oci_identity_tag" "cost_center" {
  tag_namespace_id = oci_identity_tag_namespace.project.id
  name             = "cost-center"
  is_cost_tracking = true
}
```

#### Terraform: modules/compartments/budgets.tf
```hcl
resource "oci_budget_budget" "compartment_budget" {
  for_each       = var.budget_config
  compartment_id = var.tenancy_ocid
  target_type    = "COMPARTMENT"
  targets        = [each.value.compartment_id]
  amount         = each.value.monthly_amount
  reset_period   = "MONTHLY"
}

resource "oci_budget_alert_rule" "threshold_alert" {
  for_each       = var.budget_config
  budget_id      = oci_budget_budget.compartment_budget[each.key].id
  threshold      = 80
  threshold_type = "PERCENTAGE"
  type           = "ACTUAL"
  recipients     = each.value.alert_recipients
}
```

#### Terraform: modules/compartments/quotas.tf
```hcl
resource "oci_limits_quota" "dev_quota" {
  compartment_id = var.tenancy_ocid
  name           = "dev-environment-quotas"
  statements = [
    "set compute-core quota standard-e4-core-count to 64 in compartment applications:app-name-1:dev",
    "zero gpu quota in compartment applications:app-name-1:dev"
  ]
}
```

#### OCI CLI Validation
```bash
oci iam compartment list --compartment-id $TENANCY_OCID --compartment-id-in-subtree true --all
oci iam tag-namespace list --compartment-id $TENANCY_OCID
oci budgets budget list --compartment-id $TENANCY_OCID
oci limits quota list --compartment-id $TENANCY_OCID
```

### Step 4: Validation
- Hierarchy depth <= 6 levels
- All compartments tagged
- Budget thresholds aligned
- Quota policies tested
- Cost-tracking tags enabled

### Step 5: Handoff
- Compartment OCIDs -> oci-iam, oci-networking, oci-security
- Tagging taxonomy -> all resource-creating skills
- Budget config -> oci-observability

## Security Baseline
- Cloud Guard enabled at root, inherited downward
- Security Zones on prod compartments
- No resources in root tenancy compartment
- Tag defaults enforced on compartments
