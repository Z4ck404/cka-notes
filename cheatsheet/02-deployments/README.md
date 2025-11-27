# Deployments

## Create & Scale
```bash
k create deploy nginx --image=nginx --replicas=3
k create deploy nginx --image=nginx $do > deploy.yaml  # generate yaml
k scale deploy nginx --replicas=5
```

## Update & Rollout
```bash
k set image deploy/nginx nginx=nginx:1.19              # update image
k rollout status deploy/nginx                          # watch rollout
k rollout history deploy/nginx                         # view history
k rollout undo deploy/nginx                            # rollback to previous
k rollout undo deploy/nginx --to-revision=2            # rollback to specific
k rollout restart deploy/nginx                         # restart all pods
```

## Deployment YAML (Minimal)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

## With Rolling Update Strategy
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

## Strategy Types
- **RollingUpdate** (default): gradual replacement
- **Recreate**: kill all pods, then create new
