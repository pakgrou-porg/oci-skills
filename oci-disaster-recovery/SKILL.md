---
author: Karl Miller
license: Apache 2.0
name: oci-disaster-recovery
description: "Design and implement OCI disaster recovery: Full Stack DR, cross-region replication, failover procedures. Triggers: disaster recovery, DR plan, cross-region, failover, backup, RTO, RPO, Full Stack DR."
version: "1.0.0"
tags: ["oci", "disaster-recovery", "dr", "failover", "replication", "backup"]
---

# OCI Disaster Recovery - Full Stack DR, Cross-Region, Failover

## Overview
DR planning: OCI Full Stack DR for automated failover, cross-region resource replication, backup strategies, and RTO/RPO target alignment. Covers compute, database, storage, and networking DR.

## 5-Step Process

### Step 1: Discovery
- RTO/RPO requirements per application tier
- Critical vs non-critical workload classification
- Standby region selection (paired regions)
- DR testing frequency requirements
- Compliance mandates for DR

### Step 2: Analysis
- Full Stack DR vs manual DR orchestration
- Autonomous Data Guard for database DR
- Object Storage cross-region replication
- Block Volume cross-region backup
- DNS Traffic Management for failover routing
- OKE cluster DR patterns (active-passive, active-active)

### Step 3: Creation

#### Terraform: modules/disaster-recovery/full_stack_dr.tf
```hcl
resource "oci_disaster_recovery_dr_protection_group" "primary" {
  compartment_id = var.compartment_id
  display_name   = "primary-dr-group"
  log_location { namespace = var.object_storage_namespace; bucket = "dr-logs" }

  members {
    member_id   = var.autonomous_db_ocid
    member_type = "AUTONOMOUS_DATABASE"
  }
  members {
    member_id   = var.compute_instance_ocid
    member_type = "COMPUTE_INSTANCE_MOVABLE"
  }
}

resource "oci_disaster_recovery_dr_plan" "switchover" {
  display_name         = "switchover-plan"
  dr_protection_group_id = oci_disaster_recovery_dr_protection_group.primary.id
  type                 = "SWITCHOVER"
}
```

#### Terraform: modules/disaster-recovery/cross_region_replication.tf
```hcl
resource "oci_objectstorage_replication_policy" "dr_replication" {
  namespace                    = var.object_storage_namespace
  bucket                       = var.primary_bucket_name
  name                         = "dr-replication"
  destination_bucket_name      = var.standby_bucket_name
  destination_region_name      = var.standby_region
}
```

#### DNS Failover: modules/disaster-recovery/dns_failover.tf
```hcl
resource "oci_dns_steering_policy" "failover" {
  compartment_id = var.compartment_id
  display_name   = "dr-failover-policy"
  template       = "FAILOVER"
  answers {
    name  = "primary"
    rtype = "A"
    rdata = var.primary_lb_ip
    pool  = "primary"
  }
  answers {
    name  = "standby"
    rtype = "A"
    rdata = var.standby_lb_ip
    pool  = "standby"
  }
  rules {
    rule_type = "HEALTH"
    cases { case_condition = "answer.pool == 'primary'" }
  }
  rules {
    rule_type = "PRIORITY"
    cases {
      case_condition = "answer.pool == 'primary'"
      answer_data { answer_condition = "answer.pool == 'primary'"; value = 1 }
    }
    cases {
      case_condition = "answer.pool == 'standby'"
      answer_data { answer_condition = "answer.pool == 'standby'"; value = 2 }
    }
  }
}
```

### Step 4: Validation
- DR plan dry-run successful; cross-region replication lag within RPO; DNS failover switches within RTO; standby database accessible after switchover; DR test documented

### Step 5: Handoff
- DR plan documentation; runbook for manual failover; monitoring alerts for replication lag; DR test schedule

## Security Baseline
- Standby region has identical IAM policies; encryption keys replicated to standby Vault; DR logs encrypted; failover does not bypass security controls; audit trail maintained across regions
