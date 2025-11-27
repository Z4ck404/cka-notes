# Storage

## Commands
```bash
k get pv                                               # persistent volumes
k get pvc                                              # persistent volume claims
k get storageclass                                     # storage classes
k describe pv my-pv
k describe pvc my-pvc
```

---

## emptyDir
Temporary storage, deleted when pod dies.
```yaml
volumes:
- name: cache
  emptyDir: {}
containers:
- volumeMounts:
  - name: cache
    mountPath: /tmp/cache
```

---

## hostPath
Mounts directory from host node (use sparingly).
```yaml
volumes:
- name: host-data
  hostPath:
    path: /data
    type: DirectoryOrCreate    # Directory, File, Socket, etc.
containers:
- volumeMounts:
  - name: host-data
    mountPath: /data
```

---

## PersistentVolume (PV)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain    # Retain, Delete, Recycle
  storageClassName: manual
  hostPath:                                # for testing only
    path: /mnt/data
```

---

## PersistentVolumeClaim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual                 # must match PV
```

---

## Using PVC in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /data
```

---

## StorageClass (Dynamic Provisioning)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs        # cloud specific
parameters:
  type: gp2
reclaimPolicy: Delete
allowVolumeExpansion: true
```

### PVC with StorageClass
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast                   # triggers dynamic provisioning
```

---

## Access Modes
| Mode | Abbrev | Description |
|------|--------|-------------|
| ReadWriteOnce | RWO | Single node read-write |
| ReadOnlyMany | ROX | Multiple nodes read-only |
| ReadWriteMany | RWX | Multiple nodes read-write |

---

## Reclaim Policies
| Policy | Description |
|--------|-------------|
| Retain | Keep PV after PVC deleted |
| Delete | Delete PV when PVC deleted |
| Recycle | Basic scrub (deprecated) |
