# Replicator Operator – Design Document

This document describes the internal architecture, controller logic, and reconciliation strategy of the **Replicator Operator**, which implements the `ManifestSync` Custom Resource for syncing Kubernetes manifests (Secrets, ConfigMaps and RBAC) across namespaces and clusters.

## 🎯 Goals

* 🔁 Synchronize Secrets, ConfigMaps, and RBAC
* 🚀 Support both in-cluster and remote-cluster targets
* 🧩 Allow patching, pruning, and policy-based resource handling
* ⏱ Provide real-time and interval-based sync options

## 🧱 Core Concepts

### `ManifestSync` CRD

Defines the source resource, sync targets, and policies like:

* `sourceRef`: What to replicate
* `targets`: Where to replicate (in-cluster or remote clusters)
* `syncPolicy`: Immediate or interval
* `patch`: Optional overlay
* `prune`: Cleanup logic
* `replicationOptions`: Metadata control (e.g., strip labels, ownerReferences)

### Resource Types Supported

* Secrets
* ConfigMaps
* RBAC resources (Roles, RoleBindings)

## 🛠 Architecture Overview

```text
+-------------------------------+
|         Hosting Cluster       |
|-------------------------------|
|  🧠 Replicator Operator        |
|                               |
|  - Watches ManifestSync CRs   |
|  - Fetches source resource    |
|  - Applies to targets         |
|                               |
|  📦 Cluster Credentials        |
|  - Kubeconfig secrets         |
|  - ServiceAccount tokens      |
+-------------------------------+
          /       \
         /         \   Remote API Calls
+---------------+  +------------------+
| Target Cluster|  | Target Cluster   |
| dev-cluster   |  | prod-cluster     |
+---------------+  +------------------+
```

## 🔄 Reconciliation Flow

### Triggered By:

* Changes to the `ManifestSync` resource
* Changes to the referenced `sourceRef` object
* Timer (if `syncPolicy.mode == Interval`)

### Steps:

1. **Validate `ManifestSync` spec**
2. **Fetch `sourceRef`** from the cluster
3. **Generate target manifests**

   * Apply optional `patch`
   * Sanitize metadata (remove UID, resourceVersion, etc.)
   * Respect replicationOptions like stripLabels or keepOwnerReferences
4. **Apply to targets**:

   * For `inCluster`: use standard controller-runtime client
   * For `remoteClusters`: create dynamic client from kubeconfig/token
   * Respect `resourceManagementPolicy` (Keep, Overwrite, Adopt, Sync)
5. **Update status**:

   * Record per-target sync result (Synced, Failed, etc.)
   * Set `conditions`, `lastSuccessfulSyncTime`, etc.

## ⚖️ Policies Explained

| Policy      | Behavior                                  |
| ----------- | ----------------------------------------- |
| `Keep`      | Don’t modify existing resource            |
| `Adopt`     | Take over existing resource and manage it |
| `Overwrite` | Force replace even if resource exists     |
| `Sync`      | Default: create/update as needed          |

## 🔐 Security Considerations

* Use tightly scoped `ServiceAccount` tokens for remote cluster access
* Store kubeconfigs and tokens in sealed secrets or external vaults
* Enforce RBAC on CRD and `sourceRef` access

## 🧪 Observability

* Emit events on sync success/failure
* Include condition history and timestamps in `.status`
* Add metrics (Prometheus):

  * Sync duration
  * Failure count per target
  * Resource count synced

## 🔄 Explicit Non-Goals

❌ Pull-Based Replication

We intentionally do not support pull-based (annotation-driven) replication. While this model may work for single-cluster environments, it breaks with the Replicator Operator's goals of declarative, auditable, and multi-cluster-friendly synchronization. All syncs are managed via the ManifestSync CRD for full traceability and control.