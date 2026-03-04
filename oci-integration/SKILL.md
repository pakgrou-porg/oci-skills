---
author: Karl Miller
license: Apache 2.0
name: oci-integration
description: "Design and implement OCI integration services: Streaming (Kafka-compatible), Queue, Events, Notifications, Service Connector Hub. Triggers: OCI Streaming, message queue, event rules, service connector, Kafka, pub-sub."
version: "1.0.0"
tags: ["oci", "integration", "streaming", "queue", "events", "service-connector"]
---

# OCI Integration - Streaming, Queue, Events, Notifications, Service Connector Hub

## Overview
Event-driven and messaging integration: OCI Streaming (Kafka-compatible), Queue service, Events for resource state changes, Notifications for alerting, Service Connector Hub for data pipelines.

## 5-Step Process

### Step 1: Discovery
- Messaging patterns (pub-sub, queue, stream); throughput requirements; consumer patterns; event sources; data pipeline endpoints

### Step 2: Analysis
- Streaming vs Queue tradeoffs (ordered vs at-least-once); partition strategy; retention; Service Connector source/target pairs; Kafka client compatibility

### Step 3: Creation

#### Terraform: modules/integration/streaming.tf
```hcl
resource "oci_streaming_stream_pool" "app_pool" {
  compartment_id = var.compartment_id
  name           = "${var.project_name}-stream-pool"
  kafka_settings { auto_create_topics_enable = false; log_retention_hours = 24 }
}
resource "oci_streaming_stream" "events_stream" {
  compartment_id     = var.compartment_id
  name               = "app-events"
  partitions         = var.partition_count
  stream_pool_id     = oci_streaming_stream_pool.app_pool.id
  retention_in_hours = 24
}
```

#### Terraform: modules/integration/queue.tf
```hcl
resource "oci_queue_queue" "task_queue" {
  compartment_id              = var.compartment_id
  display_name                = "task-processing-queue"
  dead_letter_queue_delivery_count = 3
  retention_in_seconds        = 345600
  visibility_in_seconds       = 30
  channel_consumption_limit   = 10
}
```

#### Terraform: modules/integration/events.tf
```hcl
resource "oci_events_rule" "db_event" {
  compartment_id = var.compartment_id
  display_name   = "db-state-change"
  is_enabled     = true
  condition      = "{\"eventType\":[\"com.oraclecloud.databaseservice.autonomous.database.backup.end\"]}"
  actions { actions { action_type = "ONS"; topic_id = var.notification_topic_id; is_enabled = true } }
}
```

#### Terraform: modules/integration/service_connector.tf
```hcl
resource "oci_sch_service_connector" "log_to_streaming" {
  compartment_id = var.compartment_id
  display_name   = "logs-to-streaming"
  source { kind = "logging"; log_sources { compartment_id = var.compartment_id; log_group_id = var.log_group_id } }
  target { kind = "streaming"; stream_id = oci_streaming_stream.events_stream.id }
}
```

### Step 4: Validation
- Stream produces/consumes; queue message lifecycle; event rules triggering; service connector flowing data; Kafka client compatibility

### Step 5: Handoff
- Stream OCIDs, queue OCIDs -> oci-containers-oke, oci-functions; event rule OCIDs -> oci-observability

## Security Baseline
- Streaming with IAM auth; queue access via policies; no plaintext sensitive data in messages; service connector scoped to specific compartments
