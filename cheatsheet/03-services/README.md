# Services

## Service Types
| Type | Description |
|------|-------------|
| **ClusterIP** | Internal only, default. Accessible within cluster |
| **NodePort** | Exposes on each node's IP at static port (30000-32767) |
| **LoadBalancer** | Cloud provider load balancer (creates NodePort + ClusterIP) |
| **ExternalName** | Maps service to external DNS name (no proxy) |

## Create Services
```bash
k expose pod nginx --port=80 --name=nginx-svc          # ClusterIP (default)
k expose deploy nginx --port=80 --type=NodePort
k create svc clusterip my-svc --tcp=80:80
k create svc nodeport my-svc --tcp=80:80 --node-port=30080
```

## Get Services
```bash
k get svc                                              # list services
k get ep                                               # endpoints (pod IPs)
k describe svc my-svc                                  # detailed info
```

## ClusterIP (default)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - port: 80
```

## NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    nodePort: 30080       # 30000-32767
```

## Headless Service (for StatefulSets)
No cluster IP assigned. Returns pod IPs directly via DNS.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - port: 80
```

## ExternalName
Maps to external DNS. No proxying, just DNS CNAME.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.example.com
```

## DNS Format
```
<service>.<namespace>.svc.cluster.local
my-svc.default.svc.cluster.local
```

## Multi-Port Service
```yaml
spec:
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
```

## Test Connectivity
```bash
k run test --image=busybox --rm -it -- wget -qO- http://my-svc
k run test --image=busybox --rm -it -- nslookup my-svc
```
