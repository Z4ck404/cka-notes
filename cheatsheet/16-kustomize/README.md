# Kustomize

Kustomize is built into kubectl for customizing Kubernetes manifests without templates.

## Basic Commands
```bash
# Apply with kustomize
kubectl apply -k ./overlay/production
kubectl apply -k .                              # current dir with kustomization.yaml

# Preview generated manifests
kubectl kustomize ./overlay/production
kubectl kustomize . > output.yaml               # save to file

# Delete resources
kubectl delete -k ./overlay/production
```

---

## Directory Structure
```
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

---

## Base kustomization.yaml
```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

---

## Overlay kustomization.yaml
```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namespace: production

namePrefix: prod-

commonLabels:
  env: production

replicas:
  - name: myapp
    count: 3

images:
  - name: nginx
    newTag: 1.25.0
```

---

## Common Transformations

### Change Replicas
```yaml
replicas:
  - name: deployment-name
    count: 5
```

### Change Image
```yaml
images:
  - name: nginx
    newName: my-registry/nginx
    newTag: v2.0.0
```

### Add Namespace
```yaml
namespace: my-namespace
```

### Add Labels/Annotations
```yaml
commonLabels:
  app: myapp
  team: backend

commonAnnotations:
  note: "managed by kustomize"
```

### Add Name Prefix/Suffix
```yaml
namePrefix: dev-
nameSuffix: -v1
```

---

## Patches

### Strategic Merge Patch
```yaml
# kustomization.yaml
patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: myapp
      spec:
        replicas: 5
```

### Patch from File
```yaml
# kustomization.yaml
patches:
  - path: replica-patch.yaml
```

```yaml
# replica-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
```

### JSON Patch
```yaml
patches:
  - target:
      kind: Deployment
      name: myapp
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
```

---

## ConfigMap & Secret Generators
```yaml
configMapGenerator:
  - name: app-config
    literals:
      - DB_HOST=localhost
      - DB_PORT=5432
  - name: app-config-file
    files:
      - config.properties

secretGenerator:
  - name: app-secret
    literals:
      - password=secret123
    type: Opaque
```

---

## Quick Reference
```bash
# Apply kustomization
kubectl apply -k <dir>

# View generated output
kubectl kustomize <dir>

# Diff before applying
kubectl diff -k <dir>

# Build and pipe to apply
kubectl kustomize . | kubectl apply -f -
```
