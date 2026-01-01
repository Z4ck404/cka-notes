# Helm

Helm is the package manager for Kubernetes. Charts are packages of pre-configured Kubernetes resources.

## Basic Commands
```bash
# Search for charts
helm search hub wordpress                       # search Artifact Hub
helm search repo nginx                          # search added repos

# Add repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# List repos
helm repo list
```

---

## Install & Manage Releases
```bash
# Install a chart
helm install my-release bitnami/nginx
helm install my-release bitnami/nginx -n my-namespace
helm install my-release bitnami/nginx --create-namespace -n my-namespace

# Install with custom values
helm install my-release bitnami/nginx -f values.yaml
helm install my-release bitnami/nginx --set replicaCount=3
helm install my-release bitnami/nginx --set image.tag=1.25.0

# Install specific version
helm install my-release bitnami/nginx --version 15.0.0

# Dry run (preview)
helm install my-release bitnami/nginx --dry-run

# Generate manifests only
helm template my-release bitnami/nginx > manifests.yaml
```

---

## List & Status
```bash
# List releases
helm list
helm list -A                                    # all namespaces
helm list -n my-namespace

# Release status
helm status my-release
helm status my-release -n my-namespace

# Release history
helm history my-release
```

---

## Upgrade & Rollback
```bash
# Upgrade release
helm upgrade my-release bitnami/nginx
helm upgrade my-release bitnami/nginx -f new-values.yaml
helm upgrade my-release bitnami/nginx --set replicaCount=5

# Install or upgrade (idempotent)
helm upgrade --install my-release bitnami/nginx

# Rollback
helm rollback my-release                        # previous revision
helm rollback my-release 1                      # specific revision
```

---

## Uninstall
```bash
helm uninstall my-release
helm uninstall my-release -n my-namespace
helm uninstall my-release --keep-history        # keep release history
```

---

## Inspect Charts
```bash
# Show chart info
helm show chart bitnami/nginx
helm show readme bitnami/nginx
helm show values bitnami/nginx                  # default values
helm show all bitnami/nginx

# Get values from release
helm get values my-release
helm get values my-release --all                # including defaults
helm get manifest my-release                    # rendered templates
```

---

## Working with Values

### values.yaml
```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.25.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi
```

### Override Values
```bash
# From file
helm install my-release bitnami/nginx -f values.yaml

# From command line
helm install my-release bitnami/nginx \
  --set replicaCount=3 \
  --set image.tag=1.25.0

# Multiple files (later overrides earlier)
helm install my-release bitnami/nginx -f values.yaml -f prod-values.yaml
```

---

## Download & Modify Charts
```bash
# Download chart
helm pull bitnami/nginx
helm pull bitnami/nginx --untar                 # extract immediately

# Install from local directory
helm install my-release ./nginx

# Package a chart
helm package ./my-chart
```

---

## Quick Reference
| Command | Description |
|---------|-------------|
| `helm repo add <name> <url>` | Add chart repository |
| `helm repo update` | Update repo index |
| `helm search repo <keyword>` | Search charts |
| `helm install <release> <chart>` | Install chart |
| `helm upgrade <release> <chart>` | Upgrade release |
| `helm rollback <release> [rev]` | Rollback to revision |
| `helm uninstall <release>` | Remove release |
| `helm list` | List releases |
| `helm status <release>` | Show release status |
| `helm get values <release>` | Get release values |
| `helm show values <chart>` | Show chart defaults |
| `helm template <release> <chart>` | Render templates locally |

---

## Common Exam Tasks
```bash
# Install chart with specific values
helm install web bitnami/nginx --set service.type=NodePort

# Upgrade with new image
helm upgrade web bitnami/nginx --set image.tag=1.26.0

# Check what changed
helm diff upgrade web bitnami/nginx -f values.yaml   # requires diff plugin

# Rollback after failed upgrade
helm rollback web 1
```
