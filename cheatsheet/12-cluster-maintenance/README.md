# Cluster Maintenance

## Cluster Upgrade (kubeadm)

> **Important:** When upgrading to a new **minor version** (e.g., 1.33 → 1.34), you must update the apt repository first. This is **not required** for patch upgrades within the same minor version (e.g., 1.34.0 → 1.34.1).

### 0. Update Package Repository (Minor Version Upgrades Only)
```bash
# Check current repo
cat /etc/apt/sources.list.d/kubernetes.list

# Edit to point to the new minor version
vim /etc/apt/sources.list.d/kubernetes.list

# Change from (example):
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /

# To:
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /

# Then update package list
apt update
```

### 1. Upgrade Control Plane
```bash
# Check available versions
apt-cache madison kubeadm

# Upgrade kubeadm
apt-get install -y kubeadm=1.34.0-1.1

# Verify kubeadm version
kubeadm version

# Plan upgrade (shows what will change)
kubeadm upgrade plan v1.34.0

# Apply upgrade
kubeadm upgrade apply v1.34.0

# Upgrade kubelet & kubectl
apt-get install -y kubelet=1.34.0-1.1 kubectl=1.34.0-1.1
systemctl daemon-reload
systemctl restart kubelet
```

### 2. Upgrade Worker Nodes
```bash
# On control plane: drain the node
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data

# SSH to worker node, then:

# Update package repo (for minor version upgrades)
vim /etc/apt/sources.list.d/kubernetes.list
# Update version in URL, then:
apt update

# Upgrade kubeadm
apt-get install -y kubeadm=1.34.0-1.1

# Upgrade node config
kubeadm upgrade node

# Upgrade kubelet & kubectl
apt-get install -y kubelet=1.34.0-1.1 kubectl=1.34.0-1.1
systemctl daemon-reload
systemctl restart kubelet

# On control plane: uncordon
kubectl uncordon node01
```

### Quick Reference: Upgrade Order
1. Update apt repo (minor version only)
2. Upgrade kubeadm
3. `kubeadm upgrade plan` → `kubeadm upgrade apply` (control plane) or `kubeadm upgrade node` (workers)
4. Upgrade kubelet & kubectl
5. Restart kubelet

---

## etcd Backup & Restore

### Local etcd for Practice (macOS/Linux)
```bash
cd cheatsheet/12-cluster-maintenance
make install   # Download etcd
make start     # Start in background
make test      # Put/get test
make backup    # Snapshot save
make restore   # Snapshot restore
make clean     # Remove all
```

---

### Backup (Kubernetes Cluster)
```bash
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db --write-out=table
```

### Backup (Local etcd - no TLS)
```bash
etcdctl --endpoints=localhost:2379 snapshot save /tmp/etcd-backup.db
etcdctl snapshot status /tmp/etcd-backup.db --write-out=table
```

### Restore (Kubernetes Cluster)
```bash
# Stop kube-apiserver (move manifest)
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# Restore to new directory
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir=/var/lib/etcd-restore

# Update etcd manifest to use new data-dir
vi /etc/kubernetes/manifests/etcd.yaml
# Change: --data-dir=/var/lib/etcd-restore
# Change: volumes hostPath to /var/lib/etcd-restore

# Restore kube-apiserver
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

### Restore (Local etcd - no TLS)
```bash
# Stop etcd first, then:
etcdutl snapshot restore /tmp/etcd-backup.db --data-dir=/tmp/etcd-restore

# Start etcd with restored data
etcd --data-dir=/tmp/etcd-restore
```

### Find etcd Certs
```bash
cat /etc/kubernetes/manifests/etcd.yaml | grep -E 'cert|key|cacert'
# or
ps aux | grep etcd | grep -E 'cert|key'
```

---

## Certificates

### Check Expiration
```bash
kubeadm certs check-expiration
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text | grep -A2 Validity
```

### Renew Certificates
```bash
kubeadm certs renew all
kubeadm certs renew apiserver
# Restart control plane components after renewal
```

### View Certificate Details
```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -subject -issuer
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text
```

---

## Node Maintenance
```bash
# Mark node unschedulable (no eviction)
kubectl cordon node01

# Drain node (evict pods)
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data

# Force drain (ignores PDBs)
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data --force

# Mark schedulable again
kubectl uncordon node01
```

---

## Important Paths
```bash
/etc/kubernetes/manifests/           # static pod manifests
/etc/kubernetes/pki/                 # cluster certificates
/etc/kubernetes/admin.conf           # admin kubeconfig
/var/lib/kubelet/config.yaml         # kubelet config
/var/lib/etcd/                       # etcd data directory
/etc/cni/net.d/                      # CNI config
```
