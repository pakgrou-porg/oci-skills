---
author: Karl Miller
license: Apache 2.0
name: oci-networking
description: "Design and implement OCI networking: VCN topologies, subnets, gateways, route tables, NSGs, security lists, load balancers, DNS, Traffic Management. Triggers: VCN design, subnet planning, NSG, load balancer, DNS, hub-spoke, DRG, private endpoints."
version: "1.0.0"
tags: ["oci", "networking", "vcn", "subnet", "nsg", "load-balancer", "dns"]
---

# OCI Networking - VCN, Subnets, Gateways, NSGs, LBs, DNS

## Overview
Designs network topologies including hub-spoke VCN, subnet segmentation, gateway routing, NSG micro-segmentation, load balancers, DNS zones, Traffic Management.

## 5-Step Process

### Step 1: Discovery
- Public-facing vs internal-only vs hybrid requirements
- On-premises/cross-region/multi-VCN connectivity
- Port/protocol requirements per tier
- DNS domain needs (public/private zones)
- Load balancing patterns (L4/L7, SSL termination)

### Step 2: Analysis
- Hub-spoke vs flat VCN tradeoffs
- CIDR planning (no overlap with on-prem)
- OCI limits: subnets/VCN, NSG rules, route entries
- Gateway requirements: IGW, NAT, SGW, DRG, LPG

### Step 3: Creation

#### Hub-Spoke Reference
```
Hub VCN (10.0.0.0/16): public-subnet 10.0.0.0/24, private 10.0.1.0/24, mgmt 10.0.2.0/24
  DRG -> Spoke 1 (10.1.0.0/16): app 10.1.0.0/24, db 10.1.1.0/24
  DRG -> Spoke 2 (10.2.0.0/16): app 10.2.0.0/24, db 10.2.1.0/24
```

#### Terraform: modules/networking/vcn.tf
```hcl
resource "oci_core_vcn" "hub" {
  compartment_id = var.networking_compartment_id
  display_name   = "hub-vcn"
  cidr_blocks    = ["10.0.0.0/16"]
  dns_label      = "hub"
}
resource "oci_core_internet_gateway" "hub_igw" {
  compartment_id = var.networking_compartment_id
  vcn_id         = oci_core_vcn.hub.id
  display_name   = "hub-igw"
}
resource "oci_core_nat_gateway" "hub_nat" {
  compartment_id = var.networking_compartment_id
  vcn_id         = oci_core_vcn.hub.id
  display_name   = "hub-nat"
}
resource "oci_core_service_gateway" "hub_sgw" {
  compartment_id = var.networking_compartment_id
  vcn_id         = oci_core_vcn.hub.id
  services { service_id = data.oci_core_services.all_services.services[0].id }
}
resource "oci_core_drg" "hub_drg" {
  compartment_id = var.networking_compartment_id
  display_name   = "hub-drg"
}
```

#### Terraform: modules/networking/nsgs.tf
```hcl
resource "oci_core_network_security_group" "lb_nsg" {
  compartment_id = var.networking_compartment_id
  vcn_id         = oci_core_vcn.hub.id
  display_name   = "lb-nsg"
}
resource "oci_core_network_security_group_security_rule" "lb_https" {
  network_security_group_id = oci_core_network_security_group.lb_nsg.id
  direction  = "INGRESS"
  protocol   = "6"
  source     = "0.0.0.0/0"
  source_type = "CIDR_BLOCK"
  tcp_options { destination_port_range { min = 443; max = 443 } }
}
resource "oci_core_network_security_group" "app_nsg" {
  compartment_id = var.networking_compartment_id
  vcn_id         = oci_core_vcn.hub.id
  display_name   = "app-nsg"
}
resource "oci_core_network_security_group" "db_nsg" {
  compartment_id = var.networking_compartment_id
  vcn_id         = oci_core_vcn.hub.id
  display_name   = "db-nsg"
}
```

#### Terraform: modules/networking/dns.tf
```hcl
resource "oci_dns_zone" "public_zone" {
  compartment_id = var.networking_compartment_id
  name           = var.public_domain
  zone_type      = "PRIMARY"
}
resource "oci_dns_zone" "private_zone" {
  compartment_id = var.networking_compartment_id
  name           = "internal.${var.public_domain}"
  zone_type      = "PRIMARY"
  scope          = "PRIVATE"
  view_id        = oci_dns_view.private_view.id
}
```

#### OCI CLI Validation
```bash
oci network vcn list --compartment-id $COMPARTMENT_OCID
oci network nsg list --compartment-id $COMPARTMENT_OCID
oci network drg list --compartment-id $COMPARTMENT_OCID
oci lb load-balancer list --compartment-id $COMPARTMENT_OCID
oci dns zone list --compartment-id $COMPARTMENT_OCID
```

### Step 4: Validation
- CIDRs non-overlapping; NSGs deny-by-default; DRG routing tested; DNS resolution verified; no broad security list ingress

### Step 5: Handoff
- VCN/subnet/NSG OCIDs -> downstream skills
- CIDR map, LB OCID, DNS zone OCIDs

## Security Baseline
- Default security lists deny-all; NSGs for micro-segmentation
- No public subnets in spokes; internet via hub NAT
- Service Gateway for OCI API access
- DB subnets private, NSG source restricted to app-tier NSG
