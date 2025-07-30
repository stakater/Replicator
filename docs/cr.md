# ManifestSync Custom Resource

This document provides a complete example and explanation of the `ManifestSync` Custom Resource (CR) for the **Replicator Operator**. This CR enables real-time or scheduled replication of Kubernetes resources (e.g., Secrets, ConfigMaps, RBAC) across namespaces and clusters.

```yaml
apiVersion: sync.stakater.com/v1alpha1
kind: ManifestSync
metadata:
  name: sync-tls-secret
  namespace: sync-operator
spec:
  # Reference to the source Kubernetes resource to be synchronized.
  sourceRef:
    apiVersion: v1
    kind: Secret
    name: tls-secret
    namespace: platform-certificates

  # Synchronization targets
  targets:
    inCluster:
      namespaces:
        - tenant-a
        - tenant-b
      namespaceSelector:
        matchLabels:
          environment: production
      resourceManagementPolicy: Sync  # Enum: Keep, Adopt, Overwrite, Sync

    remoteClusters:
      - clusterName: dev-cluster
        connection:
          type: KubeconfigSecret
          kubeconfigSecretRef:
            name: kubeconfig-dev
            namespace: sync-operator
        targetNamespace: dev-ns
        resourceManagementPolicy: Keep  # Enum: Keep, Adopt, Overwrite, Sync

      - clusterName: prod-cluster
        connection:
          type: ServiceAccountToken
          serviceAccountTokenRef:
            name: sa-token-prod
            namespace: sync-operator
        targetNamespace: prod-ns
        resourceManagementPolicy: Overwrite  # Enum: Keep, Adopt, Overwrite, Sync

  # Prune behavior
  prune:
    enabled: true
    orphanedPolicy: Delete  # Enum: Retain, Delete

  # Synchronization strategy
  syncPolicy:
    mode: Immediate  # Enum: Immediate, Interval
    # intervalSeconds: 300  # Required only if mode: Interval

  # Optional patching behavior
  patch:
    enabled: true
    patchStrategy: Merge  # Enum: Merge, JSONPatch, Replace
    patchContent:
      metadata:
        labels:
          synced-by: replicator
          compliance: enforced

  replicationOptions:
    stripLabels: true
    keepOwnerReferences: false

status:
  conditions:
    - type: Ready
      status: "True"
      reason: Synced
      message: All targets successfully synchronized.
      lastTransitionTime: "2025-07-29T19:30:00Z"
    - type: Degraded
      status: "False"
      reason: None
      message: No failures observed.
      lastTransitionTime: "2025-07-29T19:30:00Z"

  lastSuccessfulSyncTime: "2025-07-29T19:30:00Z"

  targetStatuses:
    - type: InCluster
      namespace: tenant-a
      status: Synced
      message: Secret synced successfully.
      lastSyncTime: "2025-07-29T19:30:00Z"
      error: ""

    - type: InCluster
      namespace: tenant-b
      status: Synced
      message: Secret synced successfully.
      lastSyncTime: "2025-07-29T19:30:00Z"
      error: ""

    - type: RemoteCluster
      clusterName: dev-cluster
      namespace: dev-ns
      status: Failed
      message: Failed to apply Secret. Permission denied.
      lastSyncTime: "2025-07-29T19:30:00Z"
      error: "RBAC: forbidden to create secrets in namespace dev-ns"

    - type: RemoteCluster
      clusterName: prod-cluster
      namespace: prod-ns
      status: Synced
      message: Secret synced successfully.
      lastSyncTime: "2025-07-29T19:30:00Z"
      error: ""

  sourceObservedState:
    generation: 1
    resourceVersion: "123456"
```

## 🧾 Field Descriptions

### `sourceRef`

* Defines the source Kubernetes resource to be replicated.
* Supports optional strict references (UID, resourceVersion).

### `targets.inCluster`

* Lists in-cluster namespaces or namespaceSelectors.
* `resourceManagementPolicy` controls how existing resources are handled:

  * `Keep`: Create only if missing
  * `Adopt`: Take over existing resource
  * `Overwrite`: Replace even if it exists
  * `Sync`: Update if needed, create if missing

### `targets.remoteClusters`

* Lists remote clusters by name, connection method, and target namespace.
* Supports per-cluster `resourceManagementPolicy`
* Auth via `KubeconfigSecretRef` or `ServiceAccountTokenRef`

### `prune`

* Allows deleting previously synced resources that are no longer targeted.

### `syncPolicy`

* Defines sync mode: `Immediate` or scheduled via `Interval`

### `patch`

* Allows strategic patching of synced resources
* Can add labels/annotations dynamically

### `status`

* Reflects current and historical sync state
* Reports status per target with optional error messages
