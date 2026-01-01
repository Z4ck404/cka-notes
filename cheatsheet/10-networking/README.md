# Networking

## DNS & Service Discovery
```bash
# Service DNS format
<service>.<namespace>.svc.cluster.local

# Examples
my-svc.default.svc.cluster.local
db-service.production.svc.cluster.local

# Short names (within same namespace)
my-svc
my-svc.default

# Test DNS
k run test --image=busybox --rm -it -- nslookup kubernetes.default
k exec my-pod -- cat /etc/resolv.conf
```

---

## Network Policies

### Deny All Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}              # all pods in namespace
  policyTypes:
  - Ingress
```

### Deny All Egress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

### Allow Specific Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
    - namespaceSelector:
        matchLabels:
          env: prod
    ports:
    - protocol: TCP
      port: 3306
```

### Allow Egress to External
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 10.0.0.0/8
    ports:
    - port: 443
```

---

## Ingress

### Create
```bash
k create ingress my-ing --rule="host.com/path=svc:80"
k create ingress my-ing --rule="host.com/api/*=api-svc:8080"
k get ingress
k describe ingress my-ing
```

### Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

### TLS Ingress
```yaml
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    # ...
```

---

## CNI
```bash
ls /opt/cni/bin/                                       # CNI binaries
ls /etc/cni/net.d/                                     # CNI config
cat /etc/cni/net.d/10-flannel.conflist                 # view config
k get pods -n kube-system | grep -E 'calico|flannel|weave'
```

---

## Gateway API
Gateway API is the evolution of Ingress with more expressive routing.
```yaml
# Key resources: Gateway, HTTPRoute, TLSRoute
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-route
spec:
  parentRefs:
  - name: my-gateway
  hostnames:
  - "app.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-svc
      port: 80
```
