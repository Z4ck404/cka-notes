# CKA Exam Cheat Sheet 2025

## Exam Setup (DO THIS FIRST)
```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
```

---

## Topics

| Topic | Description |
|-------|-------------|
| [01-pods](./01-pods/) | Pod creation, labels, selectors |
| [02-deployments](./02-deployments/) | Deployments, rollouts, scaling |
| [03-services](./03-services/) | Services, endpoints, networking |
| [04-configmaps-secrets](./04-configmaps-secrets/) | ConfigMaps, Secrets, env vars |
| [05-namespaces](./05-namespaces/) | Namespace management |
| [06-scheduling](./06-scheduling/) | Nodes, taints, tolerations, affinity |
| [07-multi-container](./07-multi-container/) | Init, sidecar, ambassador patterns |
| [08-storage](./08-storage/) | PV, PVC, StorageClass, volumes |
| [09-rbac](./09-rbac/) | Roles, bindings, service accounts |
| [10-networking](./10-networking/) | Network policies, Ingress, DNS |
| [11-troubleshooting](./11-troubleshooting/) | Logs, debugging, events |
| [12-cluster-maintenance](./12-cluster-maintenance/) | Upgrades, etcd, certificates |
| [13-security](./13-security/) | Security context, pod security |
| [14-workloads](./14-workloads/) | Jobs, CronJobs, DaemonSets |
| [15-jsonpath](./15-jsonpath/) | JSONPath, custom columns |
| [16-exam-tips](./16-exam-tips/) | Vim, kubectl tricks, exam strategy |

---

## Quick Reference

```bash
k run nginx --image=nginx $do > pod.yaml               # generate any yaml
k explain pod.spec.containers                          # inline docs
k api-resources                                        # list all resources
k get all -A                                           # everything everywhere
```
