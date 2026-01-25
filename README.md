# CKA Cheat Sheet 2026

> Kubernetes Certified Administrator Exam Quick Reference

## Exam Environment

> ⚠️ **Important:** Each CKA exam question requires you to **SSH into a specific node/cluster**. Always:
> - Check the **cluster context** at the start of each question
> - **SSH into the correct node** as instructed (e.g., `ssh node01`)
> - Remember that aliases/exports **don't persist** across SSH sessions
> - Use `exit` to leave a node and return to the main exam terminal

```bash
# Verify these are pre-configured before using:
alias k=kubectl                                        # usually pre-set
kubectl completion bash                                # check if available

# Useful exports (set per SSH session if needed):
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
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
| 17 | [Kustomize](cheatsheet/17-kustomize/README.md) | Manifest customization |
| 18 | [Helm](cheatsheet/18-helm/README.md) | Package manager, charts |

---

## Resources

> 📚 **Recommended resources to prepare for the CKA exam (in addition to this guide):**

| Resource | Description |
|----------|-------------|
| [KodeKloud CKA Learning Path](https://learn.kodekloud.com/user/learning-paths/cka) | Comprehensive course with hands-on labs |
| [iximiuz Labs](https://labs.iximiuz.com/) | Instant environments to practice in near real-world scenarios |
| [CKA Study Guide by David-VTUK](https://david-vtuk.github.io/CKA-StudyGuide/) | Very detailed explanations and guides |

---

## Run Locally

```bash
npm install
npx docsify serve
# Open http://localhost:3000
```
