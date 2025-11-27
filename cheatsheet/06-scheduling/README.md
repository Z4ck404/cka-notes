# Scheduling

## Node Management
```bash
k get nodes                                            # list nodes
k get nodes -o wide                                    # with IPs, OS, etc
k describe node node01                                 # detailed info
k cordon node01                                        # mark unschedulable
k uncordon node01                                      # mark schedulable
k drain node01 --ignore-daemonsets --delete-emptydir-data
```

## Node Labels
```bash
k label nodes node01 disktype=ssd                      # add label
k label nodes node01 disktype-                         # remove label
k get nodes --show-labels
k get nodes -l disktype=ssd                            # filter by label
```

## nodeSelector (simple)
```yaml
spec:
  nodeSelector:
    disktype: ssd
```

## Node Affinity (advanced)
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
            - nvme
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east-1a
```

## Pod Affinity / Anti-Affinity
```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: cache
        topologyKey: kubernetes.io/hostname
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: web
          topologyKey: kubernetes.io/hostname
```

---

## Taints & Tolerations

### Taint Nodes
```bash
k taint nodes node01 key=value:NoSchedule              # add taint
k taint nodes node01 key=value:NoSchedule-             # remove taint
k taint nodes node01 key:NoSchedule-                   # remove by key
k describe node node01 | grep Taint                    # view taints
```

### Taint Effects
```
NoSchedule        - don't schedule new pods
PreferNoSchedule  - try to avoid scheduling
NoExecute         - evict existing pods too
```

### Toleration in Pod
```yaml
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  - key: "key"
    operator: "Exists"         # matches any value
    effect: "NoSchedule"
  - operator: "Exists"         # tolerates everything
```

---

## Static Pods
```bash
# Find static pod path
cat /var/lib/kubelet/config.yaml | grep staticPodPath
# Default: /etc/kubernetes/manifests/

# Create static pod
cp pod.yaml /etc/kubernetes/manifests/

# Static pods have node name suffix
k get pods                    # nginx-node01
```
