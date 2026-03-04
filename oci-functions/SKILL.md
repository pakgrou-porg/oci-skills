---
author: Karl Miller
license: Apache 2.0
name: oci-functions
description: "Design and implement OCI Functions, API Gateway integration, and event-driven patterns. Triggers: OCI Functions, serverless, API Gateway, event-driven, function triggers."
version: "1.0.0"
tags: ["oci", "functions", "serverless", "api-gateway", "event-driven"]
---

# OCI Functions - Serverless, API Gateway, Event-Driven

## Overview
Serverless compute with OCI Functions. Covers function applications, API Gateway routing, event triggers, resource principal auth, and cold start optimization.

## 5-Step Process

### Step 1: Discovery
- Event-driven workloads; API endpoints; trigger sources (events, schedules, HTTP); language runtime (Python, Java, Node.js, Go); memory/timeout needs

### Step 2: Analysis
- Function application topology; API Gateway route design; event rule patterns; concurrency limits; cold start mitigation; resource principal vs config-based secrets

### Step 3: Creation

#### Terraform: modules/functions/main.tf
```hcl
resource "oci_functions_application" "app" {
  compartment_id = var.compartment_id
  display_name   = var.app_name
  subnet_ids     = var.subnet_ids
  config         = { "VAULT_SECRET_OCID" = var.db_secret_ocid }
}
resource "oci_functions_function" "handler" {
  application_id = oci_functions_application.app.id
  display_name   = "event-handler"
  image          = "${var.ocir_url}/${var.namespace}/${var.app_name}/handler:latest"
  memory_in_mbs  = 256
  timeout_in_seconds = 120
}
```

#### Terraform: modules/functions/api_gateway.tf
```hcl
resource "oci_apigateway_gateway" "api_gw" {
  compartment_id = var.compartment_id
  endpoint_type  = "PUBLIC"
  subnet_id      = var.public_subnet_id
  display_name   = "app-api-gateway"
}
resource "oci_apigateway_deployment" "api_deploy" {
  compartment_id = var.compartment_id
  gateway_id     = oci_apigateway_gateway.api_gw.id
  path_prefix    = "/api/v1"
  specification {
    routes {
      path    = "/process"
      methods = ["POST"]
      backend { type = "ORACLE_FUNCTIONS_BACKEND"; function_id = oci_functions_function.handler.id }
    }
  }
}
```

### Step 4: Validation
- Function invocation via fn invoke; API Gateway routing; resource principal auth to Vault; event triggers firing; cold start < 5s

### Step 5: Handoff
- Function OCIDs -> oci-devops, oci-integration; API Gateway URL -> oci-networking DNS

## Security Baseline
- Resource principal auth; secrets from Vault; functions in private subnets; API Gateway with WAF; no secrets in function config
