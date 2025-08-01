# 📘 Use Cases for Replicator Operator

## ✨ Overview

The Replicator Operator solves the common need of synchronizing Kubernetes resources across namespaces and clusters. This is particularly useful in multi-tenant architectures and hosted control plane models, where resources like Secrets, ConfigMaps, and RBAC objects must be consistently and securely replicated.

## 🧪 Use Case 1: In-Cluster Namespace Cloning

### ✅ Scenario

A platform team manages a single cluster with multiple tenant namespaces. Each tenant requires a standard set of:

* TLS Secrets
* Docker registry credentials
* ConfigMaps with common settings
* RBAC objects (Role, RoleBinding)

### ⚠️ Problem

Manually replicating resources across namespaces is error-prone and hard to maintain. GitOps workflows often lack fine-grained visibility or introduce delay.

### 💡 Solution with Replicator

* Define a `ManifestSync` per resource or group of resources
* Use `namespaceSelector` or explicit `namespaces` to target desired destinations
* Replication is automatic, patchable, auditable, and policy-controlled

### 📦 Example

```yaml
inCluster:
  namespaces:
    - tenant-a
    - tenant-b
  namespaceSelector:
    matchLabels:
      auto-sync: "true"
  resourceManagementPolicy: Sync
```

## 🌐 Use Case 2: Hosted Cluster Fan-Out

### ✅ Scenario

You operate a hosting cluster and provision tenant workloads in separate, hosted Kubernetes clusters (e.g., OpenShift Hosted Clusters, vClusters). These clusters need access to shared secrets, config, or platform policies.

### ⚠️ Problem

* You want to avoid running additional agents or operators in hosted clusters
* GitOps-based propagation may be slow or brittle
* You need a single source of truth for common resources

### 💡 Solution with Replicator

* Replicator runs **only in the hosting cluster**
* Pushes the selected manifests to hosted clusters using ServiceAccount tokens
* Resources are synchronized immediately or at defined intervals

### 📦 Example

```yaml
remoteClusters:
  clusterSelector:
    matchLabels:
      replicator-enabled: "true"
      environment: "prod"

  targetNamespaceStrategy:
    type: FromClusterAnnotation
    annotationKey: replicator.platform.io/target-namespace
    fallbackNamespace: default

  connection:
    type: ServiceAccountToken
    serviceAccountTokenSecretRef:
      name: sa-token-<clusterName>
      namespace: sync-operator

  resourceManagementPolicy: Sync
```

## ✅ Benefits

* Single source of truth
* No need for pull-based annotations
* Works across both namespace and cluster boundaries
* Declarative, real-time, secure replication
