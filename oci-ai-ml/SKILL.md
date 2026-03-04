---
author: Karl Miller
license: Apache 2.0
name: oci-ai-ml
description: "Design and implement OCI AI/ML services: Data Science, AI Services, Generative AI. Triggers: OCI Data Science, AI services, generative AI, model deployment, ML pipeline, GPU compute, inference."
version: "1.0.0"
tags: ["oci", "ai", "ml", "data-science", "generative-ai", "gpu"]
---

# OCI AI/ML - Data Science, AI Services, Generative AI

## Overview
AI/ML platform: OCI Data Science for model development/deployment, AI Services (Vision, Language, Speech, Anomaly Detection), Generative AI service, and GPU compute for training/inference.

## 5-Step Process

### Step 1: Discovery
- ML workload types (training, inference, fine-tuning); model types; GPU requirements; data pipeline sources; deployment patterns (real-time, batch); Generative AI needs (chat, embedding, summarization)

### Step 2: Analysis
- Data Science notebook shapes (GPU: VM.GPU.A10, BM.GPU.A100); model catalog options; model deployment compute; AI Services pre-built vs custom; Generative AI model selection (Cohere, Meta Llama)

### Step 3: Creation

#### Terraform: modules/ai-ml/data_science.tf
```hcl
resource "oci_datascience_project" "ml_project" {
  compartment_id = var.compartment_id
  display_name   = "${var.project_name}-ml"
}
resource "oci_datascience_notebook_session" "dev_notebook" {
  compartment_id = var.compartment_id
  project_id     = oci_datascience_project.ml_project.id
  display_name   = "dev-notebook"
  notebook_session_config_details {
    shape        = var.notebook_shape
    subnet_id    = var.private_subnet_id
    block_storage_size_in_gbs = 100
  }
}
resource "oci_datascience_model_deployment" "inference" {
  compartment_id = var.compartment_id
  project_id     = oci_datascience_project.ml_project.id
  display_name   = "model-inference"
  model_deployment_configuration_details {
    deployment_type = "SINGLE_MODEL"
    model_configuration_details {
      model_id  = var.model_ocid
      bandwidth_mbps = 10
      instance_configuration {
        instance_shape_name = var.inference_shape
      }
    }
  }
}
```

#### OCI CLI: Generative AI
```bash
oci generative-ai model list --compartment-id $COMPARTMENT_OCID
oci generative-ai dedicated-ai-cluster create --compartment-id $COMPARTMENT_OCID \
  --type FINE_TUNING --unit-count 1 --unit-shape LARGE_COHERE
```

### Step 4: Validation
- Notebook sessions launch; model deployment serving predictions; GPU shapes available; AI Services returning results; Generative AI endpoints accessible

### Step 5: Handoff
- Model deployment endpoints -> oci-containers-oke (for app integration); notebook OCIDs -> developer documentation

## Security Baseline
- Notebooks in private subnets; model artifacts encrypted; inference endpoints private; API keys in Vault; data access scoped to specific Object Storage buckets
