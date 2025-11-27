# Namespaces

## Commands
```bash
k create ns dev                                        # create namespace
k get ns                                               # list namespaces
k delete ns dev                                        # delete (and all resources!)
```

## Switch Context Namespace
```bash
k config set-context --current --namespace=dev         # switch default ns
k config view --minify | grep namespace                # check current ns
```

## Query Across Namespaces
```bash
k get pods -n kube-system                              # specific namespace
k get pods -A                                          # all namespaces
k get pods --all-namespaces                            # same as -A
k get all -A                                           # all resources, all ns
```

## Resource Quotas
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

## Limit Ranges
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: my-limits
  namespace: dev
spec:
  limits:
  - default:                  # default limits
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:           # default requests
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

## Cross-Namespace Service Access
```bash
# DNS format: <service>.<namespace>.svc.cluster.local
curl my-svc.dev.svc.cluster.local
curl my-svc.production.svc.cluster.local
```
