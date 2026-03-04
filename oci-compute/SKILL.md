---
author: Karl Miller
license: Apache 2.0
name: oci-compute
description: "Design and implement OCI compute instances, instance pools, autoscaling, and OS hardening. Triggers: OCI VM, compute instance, instance pool, autoscaling, shape selection, cloud-init, OS hardening, boot volumes."
version: "1.0.0"
tags: ["oci", "compute", "vm", "instance-pool", "autoscaling", "hardening"]
---

# OCI Compute - VMs, Instance Pools, Shapes, OS Hardening

## Overview
Compute infrastructure: shape selection, instance configs, autoscaling pools, cloud-init, OS hardening, volume management. Covers AMD (E4/E5) and Ampere (A1/A2).

## 5-Step Process

### Step 1: Discovery
- Workload type (CPU/memory/GPU-bound); availability needs; OS requirements; instance count/scaling; storage requirements

### Step 2: Analysis
- Shape families: Standard E4/E5, Optimized, GPU, Flex A1/A2; region availability; OCPU/memory calc; preemptible vs reserved

### Step 3: Creation

#### Terraform: modules/compute/main.tf
```hcl
resource "oci_core_instance" "app_instance" {
  compartment_id      = var.compartment_id
  availability_domain = var.availability_domain
  shape               = var.instance_shape
  shape_config { ocpus = var.ocpus; memory_in_gbs = var.memory_gb }
  source_details {
    source_type             = "image"
    source_id               = data.oci_core_images.oracle_linux.images[0].id
    boot_volume_size_in_gbs = var.boot_volume_size_gb
  }
  create_vnic_details {
    subnet_id        = var.subnet_id
    assign_public_ip = false
    nsg_ids          = var.nsg_ids
  }
  metadata = {
    ssh_authorized_keys = var.ssh_public_key
    user_data           = base64encode(file("${path.module}/cloud-init.yaml"))
  }
  agent_config {
    plugins_config { name = "Bastion"; desired_state = "ENABLED" }
    plugins_config { name = "Vulnerability Scanning"; desired_state = "ENABLED" }
  }
  is_pv_encryption_in_transit_enabled = true
}
```

#### Cloud-Init: modules/compute/cloud-init.yaml
```yaml
#cloud-config
package_update: true
packages: [oracle-cloud-agent, firewalld]
runcmd:
  - systemctl enable firewalld && systemctl start firewalld
  - sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
  - sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
  - systemctl restart sshd
  - setenforce 1
```

### Step 4: Validation
- Private subnets, no public IP; cloud-init success; SSH via Bastion only; autoscaling works; boot volumes encrypted; agents active

### Step 5: Handoff
- Instance OCIDs/IPs -> oci-observability; pool OCID -> oci-devops

## Security Baseline
- No public IPs; Bastion access only; SSH key-only; root login disabled; SELinux enforcing; firewalld enabled; boot volume encrypted; Vulnerability Scanning agent
