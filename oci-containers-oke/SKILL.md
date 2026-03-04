---
author: Karl Miller
license: Apache 2.0
name: oci-containers-oke
description: "Design and implement OKE clusters, node pools, K8s workloads, ingress, OCIR. Triggers: OKE cluster, Kubernetes on OCI, node pools, container deployment, OCIR, ingress controller, pod security, virtual nodes."
version: "1.0.0"
tags: ["oci", "oke", "kubernetes", "containers", "ocir", "ingress"]
---

# OCI Containers/OKE - Clusters, Node Pools, K8s Workloads, Ingress

## Overview
OKE cluster architectures: cluster config, node pool sizing, K8s workloads, ingress, OCIR image management, pod security. Primary compute for containerized workloads.

## 5-Step Process

### Step 1: Discovery
- Container workloads and resource reqs; cluster topology (single/multi/per-env); namespace strategy; ingress patterns; OCIR retention

### Step 2: Analysis
- Managed vs virtual node pools; K8s version compatibility; node shapes (AMD E4/E5, Ampere A1, GPU); flannel vs VCN-native networking; ingress options (OCI native, NGINX, Traefik); add-ons (autoscaler, metrics server, cert-manager)

### Step 3: Creation

#### Terraform: modules/oke/cluster.tf
```hcl
resource "oci_containerengine_cluster" "app_cluster" {
  compartment_id     = var.compartment_id
  kubernetes_version = var.kubernetes_version
  name               = var.cluster_name
  vcn_id             = var.vcn_id
  cluster_pod_network_options { cni_type = "OCI_VCN_IP_NATIVE" }
  endpoint_config {
    is_public_ip_enabled = false
    subnet_id            = var.api_endpoint_subnet_id
    nsg_ids              = var.api_endpoint_nsg_ids
  }
  options {
    service_lb_subnet_ids = [var.lb_subnet_id]
  }
}
```

#### Terraform: modules/oke/node_pool.tf
```hcl
resource "oci_containerengine_node_pool" "app_nodes" {
  compartment_id     = var.compartment_id
  cluster_id         = oci_containerengine_cluster.app_cluster.id
  kubernetes_version = var.kubernetes_version
  name               = "app-node-pool"
  node_shape         = var.node_shape
  node_shape_config { ocpus = var.node_ocpus; memory_in_gbs = var.node_memory_gb }
  node_config_details {
    size = var.node_count
    placement_configs {
      availability_domain = var.availability_domain
      subnet_id           = var.worker_subnet_id
    }
    nsg_ids = var.worker_nsg_ids
  }
  node_source_details {
    source_type = "IMAGE"
    image_id    = data.oci_containerengine_node_pool_option.opt.sources[0].image_id
  }
  initial_node_labels { key = "app"; value = "workload" }
}
```

#### OCIR: modules/oke/ocir.tf
```hcl
resource "oci_artifacts_container_repository" "app_repo" {
  compartment_id = var.compartment_id
  display_name   = "${var.project_name}/app"
  is_immutable   = false
  is_public      = false
}
```

#### K8s Workload Reference (deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels: { app: myapp }
  template:
    metadata:
      labels: { app: myapp }
    spec:
      containers:
        - name: app
          image: <region>.ocir.io/<namespace>/app:latest
          ports: [{ containerPort: 8080 }]
          resources:
            requests: { cpu: "250m", memory: "512Mi" }
            limits: { cpu: "1", memory: "1Gi" }
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef: { name: db-secret, key: password }
```

#### OCI CLI Validation
```bash
oci ce cluster list --compartment-id $COMPARTMENT_OCID
oci ce node-pool list --compartment-id $COMPARTMENT_OCID --cluster-id $CLUSTER_OCID
oci artifacts container repository list --compartment-id $COMPARTMENT_OCID
```

### Step 4: Validation
- API endpoint private; node pools in private subnets; K8s RBAC configured; OCIR pull secrets working; ingress routing correct; pod security policies active

### Step 5: Handoff
- Cluster OCID -> oci-devops; OCIR repo paths -> oci-devops; LB service OCID -> oci-networking

## Security Baseline
- Private API endpoint; worker nodes in private subnets; K8s secrets from OCI Vault via CSI driver; OCIR repos private; pod security standards enforced; network policies applied
