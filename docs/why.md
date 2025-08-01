# Why Replicator Operator?

This document explains the **motivation**, **context**, and **justification** for building the Replicator Operator. It outlines the existing landscape, limitations of current tools, and what makes this solution both necessary and unique.

## 📜 Background & History

In multi-tenant Kubernetes environments — especially platform-engineered setups where clusters serve multiple teams or tenants — there's often a need to **share Kubernetes resources** like:

* TLS certificates (as Secrets)
* Shared ConfigMaps (e.g., trusted CAs)
* RBAC permissions
* Default network policies

These resources may originate from:

* A centralized team (e.g., platform, security, SRE)
* A GitOps pipeline
* A provisioning process

But there is **no standard way to reliably and declaratively sync these resources** across:

* Namespaces within a cluster
* Namespaces across **different clusters**

This gap has led to repeated scripting, fragile automation, and inconsistent GitOps workarounds.

## 🧭 Existing Solutions & Their Limitations

### 1. **mittwald/kubernetes-replicator**

* ✅ Works in-cluster
* ❌ Annotation-based only
* ❌ No multi-cluster support
* ❌ No CRD to track status or control lifecycle

### 2. **Kyverno / Gatekeeper**

* ✅ Enforce & mutate resources
* ❌ Cannot sync or replicate resources

### 3. **External Secrets Operator (ESO)**

* ✅ Pulls from secret backends (e.g., Vault, AWS SM)
* ❌ Requires external source backend
* ❌ Doesn’t replicate existing K8s resources

### 4. **ArgoCD / GitOps**

* ✅ Declarative and repeatable
* ❌ Not real-time unless backed by automation
* ❌ Hard to maintain templates across clusters
* ❌ Requires Git change to update Secret, even for minor cert renewal

### 5. **SyncSet / ManifestWork (Hive / RHACM)**

* ✅ Good for initial sync
* ❌ Not reactive to source object updates
* ❌ Not flexible outside RH/Hive ecosystem

### 6. **Custom Scripts / CI Jobs**

* ✅ Fast to prototype
* ❌ Fragile, hard to maintain, no feedback loop

### 7. Crossplane

* ✅ Great for provisioning external infra (databases, buckets, etc.)
* ❌ Not designed to replicate existing in-cluster K8s resources
* ❌ No native way to watch & sync existing Secrets, ConfigMaps, RBAC
* ❌ Not push-based or real-time
* ❌ Adds overhead of XRDs and compositions for a simple sync use case

## 🚀 Why We're Building the Replicator Operator

We need a purpose-built, Kubernetes-native way to **declaratively replicate resources** both within and across clusters — securely, scalably, and observably.

### ✨ What makes this solution different:

| Feature                               | Replicator Operator           |
| ------------------------------------- | ----------------------------- |
| ✅ Real-time sync                      | Yes (via watch/reconcile)     |
| ✅ Multi-cluster support               | Yes (via kubeconfig/SA token) |
| ✅ CRD-backed lifecycle                | Yes (`ManifestSync`)          |
| ✅ Namespace selector targeting        | Yes                           |
| ✅ Patch & prune support               | Yes                           |
| ✅ Secure and RBAC-aware               | Yes                           |
| ✅ GitOps-compatible                   | Yes                           |
| ✅ Minimal hosted cluster dependencies | No agents required            |

## 🌍 Use Cases

* Distribute TLS certs to tenant namespaces and clusters
* Propagate default config for trusted proxies or root CAs
* Sync baseline RBAC policies to project namespaces
* Enable hybrid GitOps+Controller workflows (e.g., Git adds source secret, controller syncs it live)
* Share cluster-scoped config without re-declaring everywhere

## 📌 Why This is the First of Its Kind

No open-source operator today offers a:

* Generic CRD to sync **any** manifest
* **Multi-cluster** propagation model with per-target policies
* Real-time sync that works **without requiring agents** in remote clusters

The Replicator Operator is built to fill this gap with simplicity, extensibility, and zero-hosted-dependency mindset — a first in the Kubernetes ecosystem.

## ✅ Summary

The Replicator Operator exists because current tools are either:

* Too narrow in scope
* Not designed for multi-cluster
* Not truly declarative and event-driven

We’re solving this by:

* Introducing `ManifestSync` as a declarative sync CRD
* Building a scalable, secure, and platform-ready operator

> This is not just a replication tool — it's a building block for multi-tenant Kubernetes platforms.
