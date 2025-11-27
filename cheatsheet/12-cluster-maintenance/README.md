# Cluster Maintenance

## Cluster Upgrade (kubeadm)

### 1. Upgrade Control Plane
```bash
# Check available versions
apt update
apt-cache madison kubeadm

# Upgrade kubeadm
apt-get install -y kubeadm=1.30.0-1.1

# Plan upgrade
kubeadm upgrade plan

# Apply upgrade
kubeadm upgrade apply v1.30.0

# Upgrade kubelet & kubectl
apt-get install -y kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
systemctl daemon-reload
systemctl restart kubelet
```

### 2. Upgrade Worker Nodes
```bash
# On control plane: drain the node
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data

# SSH to worker node
apt-get install -y kubeadm=1.30.0-1.1
kubeadm upgrade node
apt-get install -y kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
systemctl daemon-reload
systemctl restart kubelet

# On control plane: uncordon
kubectl uncordon node01
```

---

## etcd Backup & Restore

### Backup
```bash
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db --write-out=table
```

### Restore
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
