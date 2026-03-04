---
author: Karl Miller
license: Apache 2.0
name: oci-observability
description: "Design and implement OCI observability: Logging, Monitoring, APM, Notifications, Events. Triggers: OCI logging, monitoring, alarms, APM, notifications, events, log analytics."
version: "1.0.0"
tags: ["oci", "observability", "logging", "monitoring", "apm", "notifications"]
---

# OCI Observability - Logging, Monitoring, APM, Notifications

## Overview
Observability stack: OCI Logging for log aggregation, Monitoring for metrics/alarms, APM for distributed tracing, Notifications for alerting, Events for automation triggers.

## 5-Step Process

### Step 1: Discovery
- Log sources (VCN flow logs, audit, custom app logs); metric requirements; alerting thresholds; APM instrumentation needs; notification channels (email, PagerDuty, Slack)

### Step 2: Analysis
- Log group organization; custom metric namespaces; alarm severity tiers; APM domain sizing; event rule patterns; Logging Analytics vs basic logging

### Step 3: Creation

#### Terraform: modules/observability/logging.tf
```hcl
resource "oci_logging_log_group" "app_logs" {
  compartment_id = var.compartment_id
  display_name   = "app-log-group"
}
resource "oci_logging_log" "vcn_flow_log" {
  display_name = "vcn-flow-log"
  log_group_id = oci_logging_log_group.app_logs.id
  log_type     = "SERVICE"
  configuration {
    source { category = "all"; resource = var.subnet_id; service = "flowlogs"; source_type = "OCISERVICE" }
  }
  is_enabled = true
}
```

#### Terraform: modules/observability/monitoring.tf
```hcl
resource "oci_monitoring_alarm" "high_cpu" {
  compartment_id        = var.compartment_id
  display_name          = "high-cpu-alarm"
  metric_compartment_id = var.compartment_id
  namespace             = "oci_computeagent"
  query                 = "CpuUtilization[5m].mean() > 80"
  severity              = "CRITICAL"
  destinations          = [oci_ons_notification_topic.alerts.id]
  is_enabled            = true
}
```

#### Terraform: modules/observability/notifications.tf
```hcl
resource "oci_ons_notification_topic" "alerts" {
  compartment_id = var.compartment_id
  name           = "ops-alerts"
}
resource "oci_ons_subscription" "email" {
  compartment_id = var.compartment_id
  topic_id       = oci_ons_notification_topic.alerts.id
  protocol       = "EMAIL"
  endpoint       = var.alert_email
}
```

### Step 4: Validation
- Logs flowing; alarms triggering; notifications delivered; APM traces visible; events processed

### Step 5: Handoff
- Log group OCIDs, alarm OCIDs, topic OCIDs for operational runbooks

## Security Baseline
- Audit logs always enabled; VCN flow logs on all subnets; log retention per compliance; alarm on Cloud Guard findings; no PII in log messages
