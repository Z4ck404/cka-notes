# Security

## Security Context (Pod Level)
```yaml
spec:
  securityContext:
    runAsUser: 1000                    # run as UID 1000
    runAsGroup: 3000                   # primary GID
    fsGroup: 2000                      # volume ownership GID
    runAsNonRoot: true                 # must run as non-root
```

## Security Context (Container Level)
```yaml
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsUser: 1000
      runAsNonRoot: true
      readOnlyRootFilesystem: true     # immutable filesystem
      allowPrivilegeEscalation: false
      privileged: false                # don't run as privileged
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
        drop: ["ALL"]
```

## Full Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}                       # writable /tmp
```

---

## Service Account Security
```yaml
# Disable auto-mount of SA token
spec:
  automountServiceAccountToken: false
  serviceAccountName: my-sa
```

---

## Pod Security Standards (PSS)
```yaml
# Enforce at namespace level
apiVersion: v1
kind: Namespace
metadata:
  name: my-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

### PSS Levels
| Level | Description |
|-------|-------------|
| privileged | No restrictions |
| baseline | Minimal restrictions |
| restricted | Hardened, best practices |

---

## Network Policies
See [10-networking](../10-networking/) for NetworkPolicy examples.

---

## Secrets Management
```bash
# Secrets are base64 encoded, not encrypted!
k get secret my-secret -o jsonpath='{.data.password}' | base64 -d

# Create secret
k create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

---

## RBAC Best Practices
```bash
# Use least privilege
k create role pod-reader --verb=get,list --resource=pods

# Check permissions
k auth can-i get secrets --as=system:serviceaccount:default:my-sa

# Audit who can do what
k auth can-i --list --as=jane
```

See [09-rbac](../09-rbac/) for detailed RBAC examples.

---

## API Server Audit
```bash
# Check API server audit logs
cat /var/log/kubernetes/audit.log

# Audit policy location
/etc/kubernetes/audit-policy.yaml
```
