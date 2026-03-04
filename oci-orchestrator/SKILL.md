---
author: Karl Miller
license: Apache 2.0
name: oci-orchestrator
description: "Master coordinator for Oracle Cloud Infrastructure application lifecycle. Use when planning, designing, deploying, or managing applications on OCI that span multiple service areas. Triggers: OCI application, deploy to OCI, OCI architecture, cloud infrastructure project, Oracle Cloud deployment, multi-service OCI planning. Do NOT use for single-service OCI questions."
version: "1.0.0"
tags: ["oci", "oracle-cloud", "orchestrator", "cloud-architecture", "infrastructure"]
---

# OCI Orchestrator - Master Coordinator

## Overview
Coordinates all OCI skills across the full application lifecycle. Manages dependencies, sequencing, parallel execution, and deliverable consolidation across 15 specialized OCI skills.

## Skill Registry

| # | Skill | Claude Path | AgentZero Path | Dependencies |
|---|-------|-------------|----------------|--------------|
| 1 | oci-compartments | /mnt/skills/user/oci-compartments/ | /a0/usr/skills/oci-compartments/ | None (root) |
| 2 | oci-iam | /mnt/skills/user/oci-iam/ | /a0/usr/skills/oci-iam/ | oci-compartments |
| 3 | oci-networking | /mnt/skills/user/oci-networking/ | /a0/usr/skills/oci-networking/ | oci-compartments, oci-iam |
| 4 | oci-security | /mnt/skills/user/oci-security/ | /a0/usr/skills/oci-security/ | oci-compartments, oci-iam |
| 5 | oci-compute | /mnt/skills/user/oci-compute/ | /a0/usr/skills/oci-compute/ | oci-networking, oci-iam, oci-security |
| 6 | oci-containers-oke | /mnt/skills/user/oci-containers-oke/ | /a0/usr/skills/oci-containers-oke/ | oci-networking, oci-iam, oci-security |
| 7 | oci-functions | /mnt/skills/user/oci-functions/ | /a0/usr/skills/oci-functions/ | oci-networking, oci-iam |
| 8 | oci-storage | /mnt/skills/user/oci-storage/ | /a0/usr/skills/oci-storage/ | oci-compartments, oci-iam, oci-security |
| 9 | oci-database | /mnt/skills/user/oci-database/ | /a0/usr/skills/oci-database/ | oci-networking, oci-iam, oci-security |
| 10 | oci-observability | /mnt/skills/user/oci-observability/ | /a0/usr/skills/oci-observability/ | oci-compartments, oci-iam |
| 11 | oci-devops | /mnt/skills/user/oci-devops/ | /a0/usr/skills/oci-devops/ | oci-containers-oke, oci-functions, oci-iam |
| 12 | oci-terraform | /mnt/skills/user/oci-terraform/ | /a0/usr/skills/oci-terraform/ | All skills (IaC layer) |
| 13 | oci-integration | /mnt/skills/user/oci-integration/ | /a0/usr/skills/oci-integration/ | oci-networking, oci-iam |
| 14 | oci-ai-ml | /mnt/skills/user/oci-ai-ml/ | /a0/usr/skills/oci-ai-ml/ | oci-networking, oci-iam, oci-storage |
| 15 | oci-disaster-recovery | /mnt/skills/user/oci-disaster-recovery/ | /a0/usr/skills/oci-disaster-recovery/ | All infrastructure skills |

## Standardized 5-Step Workflow (All Skills)

### Step 1: Discovery and Context Gathering
- Identify application type (containerized, serverless, VM-based, hybrid)
- Determine target OCI region(s) and availability domains
- Gather compliance requirements (Security Zones, Cloud Guard rules)
- Define tenancy structure (existing or greenfield)
- Establish success criteria

### Step 2: Research and Analysis
- Review OCI service limits and quotas for target region
- Identify shape availability and pricing
- Map cross-service dependencies
- Document assumptions, risks, fallback options

### Step 3: Creation and Documentation
- Execute skill-specific methodology
- Generate Terraform configs and/or Resource Manager stacks
- Produce OCI CLI command references
- Create architecture diagrams
- Document security baseline alignment

### Step 4: Review and Validation
- Validate Terraform plans against OCI quotas
- Verify IAM least-privilege
- Confirm NSG rules minimal and correct
- Cross-reference Vault secrets and encryption keys
- Dry-run OCI CLI commands

### Step 5: Approval and Handoff
- Consolidate deliverables for downstream skills
- Document inter-skill dependencies
- Archive Terraform state decisions
- Prepare rollback procedures

## Execution Patterns

### Pattern 1: Sequential (New Application)
Phase 1 Foundation: oci-compartments -> oci-iam -> oci-networking -> oci-security
Phase 2 Infrastructure (parallel): oci-compute | oci-containers-oke | oci-functions | oci-storage -> oci-database
Phase 3 Services: oci-integration -> oci-observability
Phase 4 Deployment: oci-devops -> oci-terraform
Phase 5 Resilience: oci-disaster-recovery
Phase 6 (optional): oci-ai-ml

### Pattern 2: Parallel (Existing Tenancy)
Track 1: oci-compartments + oci-iam (parallel)
Track 2: oci-networking + oci-security (parallel)
Sync -> oci-containers-oke or oci-compute
Track 3: oci-database + oci-storage + oci-integration (parallel)
Sync -> oci-devops -> oci-observability -> oci-terraform

### Pattern 3: Single-Service
Load specific skill directly. No orchestration needed.

## Cross-Skill Data Contract (YAML)
```yaml
skill_output:
  skill_name: "oci-networking"
  compartment_ocid: "ocid1.compartment.oc1..example"
  resources_created:
    - type: vcn
      ocid: "ocid1.vcn.oc1.iad.example"
      name: app-vcn
  terraform_module_path: "modules/networking"
  next_skills: ["oci-compute", "oci-containers-oke", "oci-database"]
```

## Security Baseline (All Skills MUST Enforce)
- Cloud Guard: Enabled with detector recipes active
- Security Zones: Applied to prod compartments
- IAM: Least-privilege; no broad manage all-resources
- Vault/KMS: All secrets in OCI Vault; zero secrets in code/env vars
- Encryption: Customer-managed keys for data-at-rest
- Bastion: Planned; no public IPs unless justified
- Audit: Logging enabled; events forwarded to observability

## PM Framework Integration
- product-architecture -> oci-orchestrator input
- product-requirements NFRs -> OCI service configs
- secure-architecture threat models -> oci-security
- deployment-planning -> oci-devops
- performance-engineering -> oci-compute/oci-containers-oke sizing

## AgentZero Notes
- Load from /a0/usr/skills/oci-*/SKILL.md
- Use call_subordinate with profile oci-engineer
- Orchestrator manages reset: true for independent contexts
- Memory tools maintain cross-skill state
