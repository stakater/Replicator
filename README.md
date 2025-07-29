# 🌀 Replicator Operator

**Replicator Operator** enables real-time, multi-cluster synchronization of Kubernetes manifests like Secrets, ConfigMaps, and RBAC resources. It is designed for platform teams operating across namespaces and clusters.

## 🚀 Features

* 🔁 Sync resources within and across clusters
* 🧩 Supports patching, pruning, and policy-based updates
* 🎯 Target by namespace name or label selector
* 🔐 Secure and RBAC-aware (no agents required in hosted clusters)
* 📜 Declarative CRD (`ManifestSync`) with full status tracking

## 🧪 Getting Started

> ⚠️ This operator is under active development.

To try it out:

1. Install the CRD and controller (via Helm, Kustomize, or raw manifests)
2. Create a `ManifestSync` resource targeting namespaces and clusters
3. Watch resources sync in real time

## 📁 Documentation

* [Why this exists](./docs/why.md) – Motivation & landscape
* [Design overview](./docs/design.md) – Architecture, controller flow
* [CR example](./docs/cr.md) – Full `ManifestSync` YAML with status

## 📌 Status

✅ MVP Design Finalized
🚧 Controller Implementation in Progress
📦 Helm Chart TBD

## ✨ License

[Apache 2.0](./LICENSE)

> Made with ❤️ for multi-tenant Kubernetes platforms by Stakater
