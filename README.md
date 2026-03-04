# Oracle Cloud Infrastructure (OCI) Skills for Agent Zero

A comprehensive collection of 16 specialized skills for designing, implementing, and managing Oracle Cloud Infrastructure applications across the full lifecycle.

## 📋 Table of Contents

- [Overview](#overview)
- [Skills Inventory](#skills-inventory)
- [Architecture](#architecture)
- [Installation](#installation)
- [Usage](#usage)
- [Examples](#examples)
- [Skill Structure](#skill-structure)
- [Dependencies](#dependencies)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

The OCI Skills collection provides Agent Zero with deep expertise in Oracle Cloud Infrastructure services. Each skill follows a standardized 5-step process and includes practical Terraform examples for Infrastructure as Code implementation.

### Key Features

- **16 Specialized Skills**: Covering all major OCI services
- **Standardized Format**: Consistent structure across all skills
- **Terraform Examples**: Real Infrastructure as Code implementations
- **Dependency Management**: Clear execution order via orchestrator
- **Production Ready**: Version 1.0.0 with comprehensive documentation
- **AI-Optimized**: Designed for AI agent orchestration and automation

## 📊 Skills Inventory

### Core Infrastructure Skills

| Skill | Description | Key Features |
|-------|-------------|--------------|
| **oci-compartments** | Tenancy structure, tagging, budgets, quotas | Compartment hierarchy, cost allocation, quota management |
| **oci-iam** | IAM policies, groups, federation, auth patterns | Least-privilege policies, dynamic groups, SAML/OIDC/SCIM |
| **oci-networking** | VCN, subnets, gateways, NSGs, LBs, DNS | Hub-spoke topologies, micro-segmentation, traffic management |
| **oci-security** | Cloud Guard, WAF, Vault/KMS, Bastion | Threat detection, encryption, secrets management, compliance |
| **oci-observability** | Logging, monitoring, APM, notifications | Metrics collection, alerting, distributed tracing |

### Compute & Runtime Skills

| Skill | Description | Key Features |
|-------|-------------|--------------|
| **oci-compute** | VMs, instance pools, autoscaling, hardening | Shape selection, cloud-init, OS hardening, AMD/Ampere support |
| **oci-containers-oke** | OKE clusters, node pools, K8s workloads | Kubernetes deployment, OCIR, pod security, virtual nodes |
| **oci-functions** | Serverless functions, API Gateway, triggers | Event-driven patterns, function triggers, serverless architecture |

### Data & Storage Skills

| Skill | Description | Key Features |
|-------|-------------|--------------|
| **oci-database** | Autonomous DB, MySQL HeatWave, NoSQL | ATP/ADW, provisioning, encryption, backups, connection patterns |
| **oci-storage** | Object Storage, Block Volumes, File Storage | Buckets, lifecycle policies, NFS, storage tiers |

### DevOps & Automation Skills

| Skill | Description | Key Features |
|-------|-------------|--------------|
| **oci-devops** | CI/CD pipelines, artifact registry, deployment | Build pipelines, blue-green/canary, artifact management |
| **oci-terraform** | Infrastructure as Code, Resource Manager | Terraform modules, state management, OCI provider |

### Integration & Advanced Services

| Skill | Description | Key Features |
|-------|-------------|--------------|
| **oci-integration** | Streaming, Queue, Events, Notifications | Kafka-compatible, pub-sub, message queue, event routing |
| **oci-ai-ml** | Data Science, AI Services, Generative AI | Model deployment, ML pipelines, GPU compute, inference |
| **oci-disaster-recovery** | Full Stack DR, cross-region replication | DR plans, failover, RTO/RPO, backup strategies |

### Master Orchestrator

| Skill | Description | Key Features |
|-------|-------------|--------------|
| **oci-orchestrator** | Master coordinator for OCI lifecycle | Dependency management, sequencing, parallel execution |

## 🏗️ Architecture

### Dependency Graph

```
oci-compartments (Root - No dependencies)
    ↓
oci-iam (Depends on: oci-compartments)
    ↓
oci-networking (Depends on: oci-compartments, oci-iam)
oci-security (Depends on: oci-compartments, oci-iam)
oci-observability (Depends on: oci-compartments, oci-iam)
    ↓
oci-compute (Depends on: oci-networking, oci-iam, oci-security)
oci-containers-oke (Depends on: oci-networking, oci-iam, oci-security)
oci-functions (Depends on: oci-networking, oci-iam)
oci-storage (Depends on: oci-compartments, oci-iam, oci-security)
oci-database (Depends on: oci-networking, oci-iam, oci-security)
oci-integration (Depends on: oci-networking, oci-iam)
oci-ai-ml (Depends on: oci-networking, oci-iam, oci-storage)
    ↓
oci-devops (Depends on: oci-containers-oke, oci-functions, oci-iam)
    ↓
oci-terraform (Depends on: ALL skills - IaC layer)
oci-disaster-recovery (Cross-cutting - depends on multiple)
```

### Standardized Skill Structure

Every skill follows this format:

```yaml
---
name: oci-skill-name
description: "Design and implement... Triggers: [keywords]"
version: "1.0.0"
tags: ["oci", "service-category", "specific-tags"]
---

# Skill Title

## Overview
Brief description of what the skill does and when to use it.

## 5-Step Process
### Step 1: Discovery
- Requirements gathering questions
- Key considerations

### Step 2: Analysis
- Technical analysis points
- Decision criteria

### Step 3: Creation
#### Terraform: modules/service/main.tf
```hcl
# Terraform code examples
resource "oci_service_resource" "example" {
  # Configuration
}
```

### Step 4: Review
- Validation checklist
- Best practices

### Step 5: Approval
- Approval criteria
- Handoff process
```

## 🚀 Installation

### Prerequisites

- Agent Zero framework installed
- Access to `/a0/usr/skills/` directory
- Python 3.x for skill execution
- Terraform (optional, for Infrastructure as Code)

### Installation Steps

1. **Clone or copy skills to Agent Zero skills directory:**

```bash
# Option 1: Copy to system skills directory
cp -r oci-skills/* /a0/usr/skills/

# Option 2: Clone directly to skills directory
git clone <repository-url> /a0/usr/skills/oci-skills
cp -r /a0/usr/skills/oci-skills/* /a0/usr/skills/
```

2. **Verify installation:**

```bash
# Count OCI skills
ls -d /a0/usr/skills/oci-* | wc -l
# Should return: 17 (including oracle-oci-brand-guidelines)

# List all OCI skills
ls -d /a0/usr/skills/oci-*
```

3. **Test a skill:**

```bash
# Check if SKILL.md exists
cat /a0/usr/skills/oci-iam/SKILL.md | head -10
```

### Verification

Run this verification script to ensure all skills are properly installed:

```bash
#!/bin/bash
echo "🔍 Verifying OCI Skills Installation..."
echo ""

# Count skills
skill_count=$(ls -d /a0/usr/skills/oci-* 2>/dev/null | wc -l)
echo "✅ OCI Skills Found: $skill_count"

# Check each skill has SKILL.md
missing_skills=0
for skill in /a0/usr/skills/oci-*/; do
  skill_name=$(basename "$skill")
  if [ ! -f "$skill/SKILL.md" ]; then
    echo "❌ Missing SKILL.md: $skill_name"
    missing_skills=$((missing_skills + 1))
  fi
done

if [ $missing_skills -eq 0 ]; then
  echo "✅ All skills have SKILL.md files"
else
  echo "❌ $missing_skills skills missing SKILL.md files"
fi

echo ""
echo "📋 Installed OCI Skills:"
ls -1 /a0/usr/skills/oci-* | xargs -n 1 basename | sort
```

## 💡 Usage

### Loading a Skill

```python
# In Agent Zero, load a skill using:
{
  "tool_name": "skills_tool:load",
  "tool_args": {
    "skill_name": "oci-iam"
  }
}
```

### Using the Orchestrator

For complex multi-service architectures, use the orchestrator:

```python
{
  "tool_name": "skills_tool:load",
  "tool_args": {
    "skill_name": "oci-orchestrator"
  }
}
```

### Chaining Skills

```python
# Example: Deploy microservices architecture
{
  "tool_name": "call_subordinate",
  "tool_args": {
    "profile": "developer",
    "message": "Use oci-networking, oci-iam, and oci-containers-oke to deploy microservices",
    "reset": "true"
  }
}
```

## 📖 Examples

### Example 1: Deploy a Web Application

**User Request**: "Deploy a web application on OCI with database backend"

**Agent Workflow**:
1. Load `oci-orchestrator` (coordinates all services)
2. Orchestrator triggers:
   - `oci-compartments` (create project compartment)
   - `oci-iam` (set up access policies)
   - `oci-networking` (create VCN and subnets)
   - `oci-compute` (deploy web servers)
   - `oci-database` (create database)
   - `oci-observability` (set up monitoring)

### Example 2: AI/ML Pipeline

**User Request**: "Build an AI/ML pipeline for image classification"

**Agent Workflow**:
1. Load `oci-ai-ml` (primary skill)
2. Load supporting skills:
   - `oci-storage` (for training data)
   - `oci-networking` (for connectivity)
   - `oci-iam` (for access control)
   - `oci-observability` (for model monitoring)

### Example 3: Disaster Recovery Setup

**User Request**: "Configure disaster recovery for critical application"

**Agent Workflow**:
1. Load `oci-disaster-recovery` (primary skill)
2. Load prerequisite skills:
   - `oci-compartments` (DR compartment)
   - `oci-database` (Data Guard configuration)
   - `oci-storage` (cross-region replication)
   - `oci-networking` (DRG setup)

## 🔧 Skill Structure Details

### YAML Frontmatter

Each skill begins with standardized metadata:

```yaml
---
name: oci-skill-name
description: "Design and implement... Triggers: [keywords]"
version: "1.0.0"
tags: ["oci", "category", "specific-tags"]
---
```

### 5-Step Process

Every skill follows this execution flow:

1. **Discovery**: Requirements gathering and context
2. **Analysis**: Technical analysis and decision criteria
3. **Creation**: Implementation with Terraform examples
4. **Review**: Validation checklist and best practices
5. **Approval**: Approval criteria and handoff

### Terraform Examples

Each skill includes practical Infrastructure as Code:

```hcl
# Example from oci-compute
resource "oci_core_instance" "app_instance" {
  compartment_id      = var.compartment_id
  availability_domain = var.availability_domain
  shape               = var.instance_shape
  
  shape_config {
    ocpus         = var.ocpus
    memory_in_gbs = var.memory_gb
  }
  
  source_details {
    source_type = "image"
    source_id   = var.image_id
  }
}
```

## 🔗 Dependencies

### Skill Dependencies

The `oci-orchestrator` skill maintains a dependency registry:

| Skill | Dependencies | Description |
|-------|--------------|-------------|
| oci-iam | oci-compartments | IAM requires compartment structure |
| oci-networking | oci-compartments, oci-iam | Networking needs IAM for access control |
| oci-compute | oci-networking, oci-iam, oci-security | Compute needs network, identity, and security |
| oci-devops | oci-containers-oke, oci-functions, oci-iam | DevOps needs runtime platforms and access |
| oci-terraform | ALL skills | Terraform orchestrates all resources |

### Execution Order

For multi-service architectures, follow this order:

1. **Foundation**: oci-compartments → oci-iam
2. **Infrastructure**: oci-networking → oci-security → oci-observability
3. **Data & Compute**: oci-storage → oci-database → oci-compute
4. **Platforms**: oci-containers-oke → oci-functions
5. **Integration**: oci-integration → oci-ai-ml
6. **Automation**: oci-devops → oci-terraform
7. **Protection**: oci-disaster-recovery

## 🤝 Contributing

### Adding New OCI Skills

1. **Create skill directory**:
   ```bash
   mkdir -p /a0/usr/skills/oci-skills/oci-new-service
   ```

2. **Create SKILL.md** following the standardized format:
   - YAML frontmatter with name, description, version, tags
   - Overview section
   - 5-step process (Discovery, Analysis, Creation, Review, Approval)
   - Terraform examples
   - Best practices

3. **Update orchestrator registry**:
   - Add skill to oci-orchestrator/SKILL.md dependency table
   - Define dependencies on other skills

4. **Test the skill**:
   ```bash
   # Load and test
   skills_tool:load {"skill_name": "oci-new-service"}
   ```

### Skill Development Guidelines

- **Use standardized format**: Follow existing skill structure
- **Include Terraform examples**: Provide practical IaC code
- **Document dependencies**: Specify prerequisite skills
- **Add trigger keywords**: Make skills easily discoverable
- **Version properly**: Use semantic versioning (currently v1.0.0)
- **Test thoroughly**: Verify skill works in isolation and with orchestrator

## 📚 Resources

### Oracle Documentation

- [OCI Documentation](https://docs.oracle.com/en-us/iaas/Content/home.htm)
- [OCI Terraform Provider](https://registry.terraform.io/providers/oracle/oci/latest)
- [OCI Architecture Center](https://docs.oracle.com/en-us/iaas/Content/General/Reference/graphicsfordiagrams.htm)
- [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)

### Agent Zero Framework

- [Agent Zero Documentation](https://github.com/frdel/agent-zero)
- [Skills Framework](https://docs.agent-zero.net/skills)
- [A2A Protocol](https://github.com/google/A2A)

## 📄 License

These skills are part of the Agent Zero framework and follow the same licensing terms.

## 🎓 Training for AI Agents

### Key Points for Subordinate Agents

When using OCI skills:

1. **Start with Orchestrator**: For multi-service scenarios, always use `oci-orchestrator`
2. **Follow Dependencies**: Skills have defined execution order - respect it
3. **Use Triggers**: Each skill has specific keywords - use them for activation
4. **Leverage Terraform**: All skills include IaC - use it for reproducibility
5. **Validate Outputs**: Use the 5-step process quality criteria
6. **Chain Skills**: Combine skills for complete architectures
7. **Test Incrementally**: Validate each skill before chaining

### Common Patterns

**Pattern 1: Web Application**
```
oci-orchestrator → oci-networking → oci-iam → oci-compute → oci-database → oci-observability
```

**Pattern 2: Serverless API**
```
oci-orchestrator → oci-iam → oci-functions → oci-api-gateway → oci-integration
```

**Pattern 3: Data Lake**
```
oci-orchestrator → oci-storage → oci-ai-ml → oci-database → oci-observability
```

## 📞 Support

For issues, questions, or contributions:
- Review skill-specific SKILL.md files
- Check the orchestrator for dependency guidance
- Test with Oracle Cloud Free Tier
- Follow the 5-step process for troubleshooting

---

