# CKA Cheat Sheet 2025

> Kubernetes Certified Administrator Exam Quick Reference

## Quick Start

```bash
# Set up aliases (do this first in exam!)
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
```

---

## Topics

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Pods](cheatsheet/01-pods/README.md) | Pod lifecycle, states, probes |
| 02 | [Deployments](cheatsheet/02-deployments/README.md) | Rolling updates, rollbacks |
| 03 | [Services](cheatsheet/03-services/README.md) | ClusterIP, NodePort, DNS |
| 04 | [ConfigMaps & Secrets](cheatsheet/04-configmaps-secrets/README.md) | Configuration management |
| 05 | [Namespaces](cheatsheet/05-namespaces/README.md) | Isolation, quotas |
| 06 | [Scheduling](cheatsheet/06-scheduling/README.md) | Affinity, taints, tolerations |
| 07 | [Multi-Container](cheatsheet/07-multi-container/README.md) | Sidecar, init containers |
| 08 | [Storage](cheatsheet/08-storage/README.md) | PV, PVC, StorageClass |
| 09 | [RBAC](cheatsheet/09-rbac/README.md) | Roles, bindings, service accounts |
| 10 | [Networking](cheatsheet/10-networking/README.md) | NetworkPolicy, Ingress |
| 11 | [Troubleshooting](cheatsheet/11-troubleshooting/README.md) | Logs, debugging, events |
| 12 | [Cluster Maintenance](cheatsheet/12-cluster-maintenance/README.md) | Upgrades, etcd, certs |
| 13 | [Security](cheatsheet/13-security/README.md) | Security contexts, PSS |
| 14 | [Workloads](cheatsheet/14-workloads/README.md) | Jobs, CronJobs, DaemonSets |
| 15 | [JSONPath](cheatsheet/15-jsonpath/README.md) | Output formatting |
| 16 | [Exam Tips](cheatsheet/16-exam-tips/README.md) | Time-saving tricks |

---

## Run Locally

```bash
npm install
npx docsify serve
# Open http://localhost:3000
```
