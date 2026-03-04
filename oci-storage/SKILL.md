---
author: Karl Miller
license: Apache 2.0
name: oci-storage
description: "Design and implement OCI storage: Object Storage, Block Volumes, File Storage. Triggers: object storage, buckets, block volumes, file storage, NFS, storage tiers, lifecycle policies."
version: "1.0.0"
tags: ["oci", "storage", "object-storage", "block-volumes", "file-storage"]
---

# OCI Storage - Object Storage, Block Volumes, File Storage

## Overview
Storage architecture across Object Storage (S3-compatible), Block Volumes, and File Storage (NFS). Covers lifecycle policies, encryption, replication, and tiering.

## 5-Step Process

### Step 1: Discovery
- Data types and access patterns; capacity requirements; performance (IOPS, throughput); retention/lifecycle; cross-region replication needs

### Step 2: Analysis
- Object Storage tiers (Standard, Infrequent, Archive); Block Volume performance tiers (Lower/Balanced/Higher); File Storage sizing; pre-authenticated requests vs PAR

### Step 3: Creation

#### Terraform: modules/storage/object_storage.tf
```hcl
resource "oci_objectstorage_bucket" "app_bucket" {
  compartment_id = var.compartment_id
  namespace      = var.object_storage_namespace
  name           = "${var.project_name}-data"
  access_type    = "NoPublicAccess"
  storage_tier   = "Standard"
  versioning     = "Enabled"
  kms_key_id     = var.encryption_key_id
  object_events_enabled = true
  auto_tiering    = "InfrequentAccess"
}

resource "oci_objectstorage_object_lifecycle_policy" "cleanup" {
  namespace  = var.object_storage_namespace
  bucket     = oci_objectstorage_bucket.app_bucket.name
  rules {
    name        = "archive-old-objects"
    action      = "ARCHIVE"
    time_amount = 90
    time_unit   = "DAYS"
    is_enabled  = true
  }
}
```

#### Terraform: modules/storage/block_volumes.tf
```hcl
resource "oci_core_volume" "data_volume" {
  compartment_id      = var.compartment_id
  availability_domain = var.availability_domain
  display_name        = "app-data-volume"
  size_in_gbs         = var.volume_size_gb
  vpus_per_gb         = 20  # Balanced performance
  kms_key_id          = var.encryption_key_id
}
```

### Step 4: Validation
- Buckets private; encryption with CMK; lifecycle policies active; block volume IOPS meeting requirements; replication configured

### Step 5: Handoff
- Bucket names/namespaces -> oci-containers-oke, oci-functions; volume OCIDs -> oci-compute

## Security Baseline
- All buckets NoPublicAccess; customer-managed encryption keys; versioning enabled on critical buckets; object events for audit; no pre-authenticated requests without expiration
