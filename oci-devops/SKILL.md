---
author: Karl Miller
license: Apache 2.0
name: oci-devops
description: "Design and implement OCI DevOps: build pipelines, deployment pipelines, artifact registry, deployment strategies. Triggers: OCI DevOps, CI/CD, build pipeline, deployment pipeline, artifact registry, blue-green, canary deployment."
version: "1.0.0"
tags: ["oci", "devops", "cicd", "pipeline", "artifact-registry", "deployment"]
---

# OCI DevOps - Pipelines, Artifact Registry, Deployment Strategies

## Overview
CI/CD with OCI DevOps: build pipelines for compilation/testing/image building, deployment pipelines for OKE/Functions/instance group rollouts, artifact management, and deployment strategies.

## 5-Step Process

### Step 1: Discovery
- Source repositories (OCI Code Repo, GitHub, GitLab); build requirements; deployment targets (OKE, Functions, instance groups); deployment strategy (rolling, blue-green, canary); approval gates

### Step 2: Analysis
- Build runner shapes; build spec stages; deployment pipeline topology; artifact types (container image, generic); trigger patterns (push, tag, manual)

### Step 3: Creation

#### Terraform: modules/devops/project.tf
```hcl
resource "oci_devops_project" "app_project" {
  compartment_id = var.compartment_id
  name           = var.project_name
  notification_config { topic_id = var.notification_topic_id }
}
```

#### Terraform: modules/devops/build_pipeline.tf
```hcl
resource "oci_devops_build_pipeline" "app_build" {
  project_id   = oci_devops_project.app_project.id
  display_name = "app-build-pipeline"
}
resource "oci_devops_build_pipeline_stage" "build" {
  build_pipeline_id = oci_devops_build_pipeline.app_build.id
  display_name      = "build-stage"
  build_pipeline_stage_type = "BUILD"
  build_spec_file           = "build_spec.yaml"
  build_runner_shape_config { build_runner_type = "CUSTOM"; memory_in_gbs = 8; ocpus = 2 }
  build_source_collection {
    items { connection_type = "GITHUB"; repository_url = var.github_repo_url; branch = "main"; name = "source" }
  }
}
resource "oci_devops_build_pipeline_stage" "deliver" {
  build_pipeline_id = oci_devops_build_pipeline.app_build.id
  display_name      = "deliver-artifacts"
  build_pipeline_stage_type = "DELIVER_ARTIFACT"
  deliver_artifact_collection {
    items { artifact_id = oci_devops_deploy_artifact.container_image.id; artifact_name = "app-image" }
  }
  build_pipeline_stage_predecessor_collection {
    items { id = oci_devops_build_pipeline_stage.build.id }
  }
}
```

#### Terraform: modules/devops/deploy_pipeline.tf
```hcl
resource "oci_devops_deploy_pipeline" "app_deploy" {
  project_id   = oci_devops_project.app_project.id
  display_name = "app-deploy-pipeline"
}
resource "oci_devops_deploy_stage" "oke_deploy" {
  deploy_pipeline_id = oci_devops_deploy_pipeline.app_deploy.id
  display_name       = "deploy-to-oke"
  deploy_stage_type  = "OKE_DEPLOYMENT"
  oke_cluster_deploy_environment_id = oci_devops_deploy_environment.oke_env.id
  kubernetes_manifest_deploy_artifact_ids = [oci_devops_deploy_artifact.k8s_manifest.id]
  rollback_policy { policy_type = "AUTOMATED_STAGE_ROLLBACK_POLICY" }
}
```

### Step 4: Validation
- Build pipeline compiles and produces artifacts; deployment pipeline deploys to OKE/Functions; rollback tested; approval gates working; triggers firing

### Step 5: Handoff
- Pipeline OCIDs for operational documentation; build spec templates for developers

## Security Baseline
- Build secrets from Vault; OCIR images scanned; deployment approval gates for prod; rollback policies on all prod stages; audit trail on all deployments
