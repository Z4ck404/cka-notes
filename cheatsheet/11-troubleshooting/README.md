# Troubleshooting

## Pod Logs
```bash
k logs pod-name                                        # current logs
k logs pod-name -f                                     # follow/stream
k logs pod-name --previous                             # crashed container logs
k logs pod-name -c container-name                      # specific container
k logs pod-name --all-containers                       # all containers
k logs pod-name --since=1h                             # last hour
k logs pod-name --tail=100                             # last 100 lines
k logs -l app=nginx                                    # by label selector
```

## Exec Into Pod
```bash
k exec -it pod-name -- sh
k exec -it pod-name -- /bin/bash
k exec -it pod-name -c container-name -- sh            # specific container
k exec pod-name -- cat /etc/resolv.conf                # run single command
k exec pod-name -- env                                 # view env vars
```

## Describe & Events
```bash
k describe pod pod-name                                # detailed info + events
k describe node node01
k get events                                           # all events
k get events --sort-by='.lastTimestamp'                # sorted by time
k get events --field-selector type=Warning             # only warnings
k get events -w                                        # watch events
```

## Pod Status Meanings
| Status | Meaning |
|--------|---------|
| Pending | Waiting to be scheduled |
| ContainerCreating | Pulling image / starting |
| Running | At least one container running |
| Completed | All containers exited 0 |
| Error | Container exited non-zero |
| CrashLoopBackOff | Keeps crashing, backing off |
| ImagePullBackOff | Can't pull image |
| ErrImagePull | Image pull failed |

---

## Troubleshoot Services
```bash
k get endpoints my-svc                                 # check if pods registered
k describe svc my-svc                                  # check selector
k get pods -l app=myapp                                # matching pods?
k run test --image=busybox --rm -it -- wget -qO- http://my-svc
k run test --image=busybox --rm -it -- nc -zv my-svc 80
```

---

## Troubleshoot Nodes
```bash
k get nodes                                            # node status
k describe node node01                                 # conditions, allocatable
k top nodes                                            # resource usage
```

### On the Node (SSH)
```bash
systemctl status kubelet                               # kubelet status
systemctl restart kubelet
journalctl -u kubelet -f                               # kubelet logs
journalctl -u kubelet --since "10 minutes ago"

cat /var/lib/kubelet/config.yaml                       # kubelet config
ls /etc/kubernetes/manifests/                          # static pod manifests

crictl ps                                              # list containers
crictl logs <container-id>                             # container logs
crictl inspect <container-id>                          # container details
```

---

## Troubleshoot Control Plane
```bash
# Check control plane pods
k get pods -n kube-system

# Check static pod manifests
ls /etc/kubernetes/manifests/
cat /etc/kubernetes/manifests/kube-apiserver.yaml

# API server logs
k logs -n kube-system kube-apiserver-master
crictl logs <apiserver-container-id>

# etcd logs
k logs -n kube-system etcd-master

# Scheduler/Controller Manager
k logs -n kube-system kube-scheduler-master
k logs -n kube-system kube-controller-manager-master
```

---

## Resource Usage
```bash
k top nodes                                            # node CPU/memory
k top pods                                             # pod CPU/memory
k top pods --containers                                # per container
k top pods -A --sort-by=memory                         # sort by memory
```

---

## Quick Debug Pod
```bash
# Temporary debug pod
k run debug --image=busybox --rm -it -- sh
k run debug --image=nicolaka/netshoot --rm -it -- bash # network tools

# Debug in same network namespace as pod
k debug -it pod-name --image=busybox --target=container-name
```
