---
author: Karl Miller
license: Apache 2.0
name: oci-iam
description: "Design and implement OCI IAM policies, groups, dynamic groups, federation, and authentication patterns. Triggers: IAM policies, OCI permissions, user groups, dynamic groups, identity federation, IDCS, identity domains, service principals, API keys, auth tokens."
version: "1.0.0"
tags: ["oci", "iam", "identity", "access-management", "policies", "federation"]
---

# OCI IAM - Policies, Groups, Dynamic Groups, Federation

## Overview
Designs least-privilege IAM policies, group structures, dynamic groups for service-to-service auth, and identity federation. All OCI skills depend on IAM.

## 5-Step Process

### Step 1: Discovery
- User roles and team structures
- Service accounts and automation needs
- Federation requirements (SAML, OIDC, SCIM)
- Cross-compartment access patterns

### Step 2: Analysis
- IAM verb hierarchy: inspect < read < use < manage
- Resource types per role permission level
- Dynamic group matching rules for OCI services
- API key vs instance principal vs resource principal patterns

### Step 3: Creation

#### Groups: modules/iam/groups.tf
```hcl
resource "oci_identity_group" "admins" {
  compartment_id = var.tenancy_ocid
  name           = "oci-admins"
  description    = "Tenancy administrators"
}
resource "oci_identity_group" "network_admins" {
  compartment_id = var.tenancy_ocid
  name           = "network-admins"
}
resource "oci_identity_group" "developers" {
  compartment_id = var.tenancy_ocid
  name           = "developers"
}
resource "oci_identity_group" "security_admins" {
  compartment_id = var.tenancy_ocid
  name           = "security-admins"
}
resource "oci_identity_group" "devops_engineers" {
  compartment_id = var.tenancy_ocid
  name           = "devops-engineers"
}
```

#### Dynamic Groups: modules/iam/dynamic_groups.tf
```hcl
resource "oci_identity_dynamic_group" "oke_clusters" {
  compartment_id = var.tenancy_ocid
  name           = "oke-clusters"
  matching_rule  = "ALL {resource.type = 'cluster', resource.compartment.id = '${var.app_compartment_id}'}"
}
resource "oci_identity_dynamic_group" "functions" {
  compartment_id = var.tenancy_ocid
  name           = "fn-functions"
  matching_rule  = "ALL {resource.type = 'fnfunc', resource.compartment.id = '${var.app_compartment_id}'}"
}
resource "oci_identity_dynamic_group" "devops_pipelines" {
  compartment_id = var.tenancy_ocid
  name           = "devops-pipelines"
  matching_rule  = "ALL {resource.type = 'devopsbuildpipeline', resource.compartment.id = '${var.devops_compartment_id}'}"
}
resource "oci_identity_dynamic_group" "instances" {
  compartment_id = var.tenancy_ocid
  name           = "app-instances"
  matching_rule  = "ALL {instance.compartment.id = '${var.app_compartment_id}'}"
}
```

#### Policies: modules/iam/policies.tf
```hcl
resource "oci_identity_policy" "network_admin" {
  compartment_id = var.tenancy_ocid
  name           = "network-admin-policy"
  statements = [
    "Allow group network-admins to manage virtual-network-family in compartment shared-services:networking",
    "Allow group network-admins to manage load-balancers in compartment shared-services:networking",
    "Allow group network-admins to read all-resources in tenancy"
  ]
}
resource "oci_identity_policy" "developer" {
  compartment_id = var.tenancy_ocid
  name           = "developer-policy"
  statements = [
    "Allow group developers to manage functions-family in compartment applications",
    "Allow group developers to use virtual-network-family in compartment shared-services:networking",
    "Allow group developers to manage repos in compartment platform:devops"
  ]
}
resource "oci_identity_policy" "oke_policy" {
  compartment_id = var.tenancy_ocid
  name           = "oke-cluster-policy"
  statements = [
    "Allow dynamic-group oke-clusters to use secret-family in compartment shared-services:security",
    "Allow dynamic-group oke-clusters to manage objects in compartment data:storage"
  ]
}
resource "oci_identity_policy" "functions_policy" {
  compartment_id = var.tenancy_ocid
  name           = "functions-policy"
  statements = [
    "Allow dynamic-group fn-functions to use secret-family in compartment shared-services:security",
    "Allow dynamic-group fn-functions to manage objects in compartment data:storage"
  ]
}
```

#### OCI CLI Validation
```bash
oci iam group list --compartment-id $TENANCY_OCID --all
oci iam dynamic-group list --compartment-id $TENANCY_OCID --all
oci iam policy list --compartment-id $TENANCY_OCID --all
```

### Step 4: Validation
- No manage all-resources in tenancy (except break-glass admin)
- Dynamic groups scoped to specific compartments
- Instance principal auth tested
- No overlapping/conflicting policies

### Step 5: Handoff
- Group names/OCIDs for user assignment
- Dynamic group names -> oci-containers-oke, oci-functions, oci-devops
- Policy OCIDs for audit

## Security Baseline
- No manage all-resources except designated admin
- Dynamic groups compartment-scoped
- Resource principals preferred over API keys
- API keys in OCI Vault (see oci-security)
