# Multi-Container Pods

## Patterns Overview
| Pattern | Purpose | Runs |
|---------|---------|------|
| **Init Container** | Setup before app starts | Before main, must complete |
| **Sidecar** | Helper running alongside | With main container |
| **Ambassador** | Proxy to external services | With main container |
| **Adapter** | Transform output | With main container |

---

## Init Containers
Runs before main containers, must complete successfully.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'until nc -z mydb 3306; do echo waiting; sleep 2; done']
  - name: init-config
    image: busybox
    command: ['sh', '-c', 'wget -O /config/app.conf http://config-server/app.conf']
    volumeMounts:
    - name: config
      mountPath: /config
  containers:
  - name: app
    image: myapp
    volumeMounts:
    - name: config
      mountPath: /config
  volumes:
  - name: config
    emptyDir: {}
```

---

## Sidecar Pattern
Runs alongside main container, shares volumes/network.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logging
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  - name: log-collector                                # sidecar
    image: busybox
    command: ['sh', '-c', 'tail -f /var/log/nginx/access.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  volumes:
  - name: logs
    emptyDir: {}
```

### Common Sidecar Use Cases
- Log shipping (fluentd, filebeat)
- Metrics collection (prometheus exporter)
- Service mesh proxy (envoy, istio)

---

## Ambassador Pattern
Proxy container for external services.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-ambassador
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: DB_HOST
      value: "localhost"                               # talks to ambassador
    - name: DB_PORT
      value: "3306"
  - name: ambassador                                   # proxy to external DB
    image: ambassador-proxy
    ports:
    - containerPort: 3306
    env:
    - name: BACKEND_HOST
      value: "external-db.example.com"
```

---

## Adapter Pattern
Transform output format (e.g., logs to JSON).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-adapter
spec:
  containers:
  - name: app
    image: myapp
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
  - name: adapter                                      # transforms logs
    image: log-adapter
    command: ['sh', '-c', 'tail -f /var/log/app/app.log | jq -R "..."']
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
  volumes:
  - name: logs
    emptyDir: {}
```

---

## Debugging Multi-Container Pods
```bash
k logs pod-name -c container-name                      # specific container
k logs pod-name --all-containers                       # all containers
k exec -it pod-name -c container-name -- sh            # exec into specific
k describe pod pod-name                                # see all container status
```
