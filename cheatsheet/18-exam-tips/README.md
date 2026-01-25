# Exam Tips ⭐

## 🖥️ Exam Environment

> ⚠️ **Important:** Each question requires you to **SSH into a specific node/cluster**.

- Check the **cluster context** at the start of each question
- **SSH into the correct node** as instructed (e.g., `ssh node01`)
- Aliases/exports **don't persist** across SSH sessions
- Use `exit` to return to the main terminal

```bash
# Verify these are pre-configured:
alias k=kubectl                        # usually pre-set

# Useful exports (set per SSH session if needed):
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
```

---

## 📖 Kubernetes Docs Navigation

> 🔑 **The docs are open during the exam!** Learn to navigate them quickly.

**Best resource:** [kubernetes.io/docs/tasks/](https://kubernetes.io/docs/tasks/) - Ready-to-copy YAML examples

| What you need | Search for |
|---------------|------------|
| Probes | `configure liveness readiness` |
| PV/PVC | `configure persistent volume` |
| RBAC | `configure service account` |
| NetworkPolicy | `declare network policy` |
| Security Context | `security context pod` |
| Resource Limits | `assign cpu memory` |
| Taints/Tolerations | `taint toleration` |

**Allowed documentation:**
- kubernetes.io/docs
- kubernetes.io/blog  
- github.com/kubernetes

---

## 🎯 Exam Strategy

1. **Read all questions first** - Identify easy wins
2. **Do easy questions first** - Bank points quickly
3. **Use imperative commands** - Faster than writing YAML
4. **Generate YAML, then modify** - Use `$do` alias
5. **Verify after each question** - Don't assume it worked
6. **Flag difficult questions** - Return if time permits
7. **Don't over-engineer** - Do exactly what's asked

### Common Mistakes to Avoid
- **Forgetting namespace** - Use `-n` or set context
- **Wrong YAML indentation** - Use `k explain` to verify
- **Typos in labels/selectors** - Copy-paste when possible
- **Not reading carefully** - Check namespace, names, values

---

## ✅ Validate Your Work

**Always verify before moving to the next question!**

| Task | Verification | Expected |
|------|--------------|----------|
| Pods/Deployments | `k get po -o wide` | STATUS `Running`, READY `1/1` |
| Services | `k get ep <svc>` | Endpoints exist |
| RBAC | `k auth can-i <verb> <resource> --as <user>` | `yes` |
| Storage | `k get pv,pvc` | STATUS `Bound` |
| NetworkPolicy | `k exec <pod> -- curl <target>` | Works/blocked as expected |
| Cluster Upgrade | `k get nodes` | VERSION matches target |

```bash
# Quick connectivity test
k run test --rm -it --image=busybox -- wget -qO- http://<svc>
```

---

## ⚡ Essential Commands

### Generate YAML
```bash
k run nginx --image=nginx $do > pod.yaml
k create deploy nginx --image=nginx $do > deploy.yaml
k create svc clusterip my-svc --tcp=80:80 $do > svc.yaml
k create job my-job --image=busybox $do -- echo hi > job.yaml
k create cm my-cm --from-literal=k=v $do > cm.yaml
k create secret generic my-s --from-literal=k=v $do > secret.yaml
k create sa my-sa $do > sa.yaml
k create role my-role --verb=get --resource=pods $do > role.yaml
k create ingress my-ing --rule="h.com/=svc:80" $do > ingress.yaml
```

### kubectl explain
```bash
k explain pod.spec.containers.livenessProbe
k explain pv.spec --recursive | grep -A5 capacity
k explain pod --recursive | less
k api-resources | grep <resource>
```

### Debugging
```bash
k run test --image=busybox --rm -it -- sh              # temp debug pod
k get events --sort-by='.lastTimestamp' | tail -20     # recent events
k logs <pod> --previous                                # crashed container
k delete pod <name> $now                               # force delete
```

### Context & Namespace
```bash
kubectl config use-context <context-name>              # switch context
k config set-context --current --namespace=<ns>        # set default ns
k config current-context                               # verify context
```

---

## 📂 Important Paths

```bash
/etc/kubernetes/manifests/          # static pods (control plane)
/etc/kubernetes/pki/                # cluster certificates
/etc/kubernetes/pki/etcd/           # etcd certificates
/var/lib/kubelet/config.yaml        # kubelet config
/var/lib/etcd/                      # etcd data directory
/etc/cni/net.d/                     # CNI config
```

### etcd Backup (memorize this!)
```bash
ETCDCTL_API=3 etcdctl snapshot save /backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### Troubleshoot Nodes
```bash
systemctl status kubelet
journalctl -u kubelet -f
crictl ps
```

---

## 📝 Vim Quick Reference

```bash
# Essential settings
:set number paste tabstop=2 shiftwidth=2 expandtab

# Navigation
gg / G                    # top / bottom
:42                       # go to line 42
/pattern  n/N             # search, next/prev

# Editing
dd / yy / p               # delete / copy / paste line
u / Ctrl+r                # undo / redo

# Save & Quit
:wq                       # save and quit
:q!                       # quit without saving
```

---

## Good Luck! 🎯

- Stay calm
- Manage your time
- Trust your preparation
