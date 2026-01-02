# Exam Tips

## ⚠️ Validate Before Moving On!

**Always verify your work before moving to the next question.**

| Task | Verification Command | Expected Output |
|------|---------------------|-----------------|
| **Workloads** | `k get deploy,po` | READY `1/1`, STATUS `Running` |
| **Services** | `k exec <pod> -- curl <svc-ip>` | HTTP `200 OK` response |
| **RBAC** | `k auth can-i <verb> <resource> --as <user>` | `yes` |
| **Storage** | `k get pv,pvc` | STATUS `Bound` (not `Pending`) |
| **HPA/VPA** | `k describe hpa <name>` | Check `Targets` and `Replicas` |
| **Cluster Upgrade** | `k get nodes` | VERSION matches target |
| **NetworkPolicy** | `k exec <pod> -- curl <target>` | Connection works/blocked as expected |
| **Secrets/ConfigMaps** | `k exec <pod> -- env \| grep <KEY>` | Value is set correctly |

### Quick Validation Commands
```bash
# Check pod is running
k get po <name> -o wide

# Check service endpoints exist
k get ep <svc-name>

# Test service connectivity
k run test --rm -it --image=busybox -- wget -qO- http://<svc>

# Verify RBAC
k auth can-i get pods --as system:serviceaccount:ns:sa-name

# Check PVC is bound
k get pvc <name> -o jsonpath='{.status.phase}'

# Verify node version after upgrade
k get nodes -o wide
```

---

## First Things First
```bash
# Set these up immediately!
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
```

---

## Vim Essentials
```bash
# In ~/.vimrc (or type in vim)
:set number                            # line numbers
:set tabstop=2                         # tab = 2 spaces
:set shiftwidth=2                      # indent = 2 spaces
:set expandtab                         # tabs to spaces
:set paste                             # paste without auto-indent

# Navigation
gg                                     # go to top
G                                      # go to bottom
:42                                    # go to line 42
/pattern                               # search forward
n / N                                  # next/prev match

# Editing
dd                                     # delete line
yy                                     # copy line
p                                      # paste below
P                                      # paste above
u                                      # undo
Ctrl+r                                 # redo
:%s/old/new/g                          # replace all

# Save & Quit
:w                                     # save
:q                                     # quit
:wq                                    # save and quit
:q!                                    # quit without saving
```

---

## kubectl Generate YAML
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

---

## kubectl explain (Your Best Friend)
```bash
k explain pod.spec.containers
k explain pod.spec.containers.livenessProbe
k explain pod.spec.containers.securityContext.capabilities
k explain pv.spec --recursive | grep -A5 capacity
k explain deploy.spec.strategy
k explain deploy.spec.template.spec.imagePullSecrets   # list of secret refs
k explain pod --recursive | less                       # full reference

# Find API group/version for any resource
k api-resources | grep <resource>
```

---

## Quick Debugging
```bash
# Temp pod for testing
k run test --image=busybox --rm -it -- sh
k run test --image=busybox --rm -it -- wget -qO- http://my-svc
k run test --image=busybox --rm -it -- nslookup my-svc

# Copy existing pod to edit
k get pod my-pod -o yaml > debug.yaml
# Edit debug.yaml, then apply

# Check events
k get events --sort-by='.lastTimestamp' | tail -20
```

---

## Time-Saving Tricks
```bash
# Force delete stuck pods
k delete pod my-pod $now

# Get all resource types
k api-resources

# Check what you can do
k auth can-i --list

# Diff before apply
k diff -f manifest.yaml

# Count resources
k get pods --no-headers | wc -l
```

---

## Common Mistakes to Avoid
1. **Forgetting namespace** - Use `-n` or set context
2. **Wrong indentation in YAML** - Use `k explain` to verify structure
3. **Typos in labels/selectors** - Copy-paste when possible
4. **Not reading question carefully** - Check namespace, names, values
5. **Spending too long on one question** - Flag and move on

---

## Context Switching
```bash
# Exam gives you context commands - USE THEM
kubectl config use-context <context-name>

# Verify current context
k config current-context
k config get-contexts

# Set default namespace for context
k config set-context --current --namespace=my-ns
```

---

## Documentation Allowed
- kubernetes.io/docs
- kubernetes.io/blog
- github.com/kubernetes

### Useful Bookmarks
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [Tasks (How-to)](https://kubernetes.io/docs/tasks/)

---

## Exam Strategy
1. **Read all questions first** - Note easy wins
2. **Do easy questions first** - Build confidence, bank points
3. **Use imperative commands** - Faster than writing YAML
4. **Generate YAML, then modify** - Use `$do` alias
5. **Verify after each question** - Don't assume it worked
6. **Flag difficult questions** - Come back if time permits
7. **Don't over-engineer** - Do exactly what's asked

---

## Useful Paths to Remember
```bash
/etc/kubernetes/manifests/             # static pods (control plane)
/etc/kubernetes/pki/                   # certificates
/etc/kubernetes/pki/etcd/              # etcd certificates
/var/lib/kubelet/config.yaml           # kubelet config
/var/lib/etcd/                         # etcd data directory
/etc/cni/net.d/                        # CNI config
```

---

## Quick Reference
```bash
# Static pods location
/etc/kubernetes/manifests/

# etcd backup
ETCDCTL_API=3 etcdctl snapshot save /backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Check kubelet
systemctl status kubelet
journalctl -u kubelet

# Container runtime
crictl ps
crictl logs <container-id>
```

---

## Good Luck! 🎯
- Stay calm
- Manage your time  
- Trust your preparation
