# JSONPath & Output Formatting

## Basic JSONPath
```bash
# Get all pod names
k get pods -o jsonpath='{.items[*].metadata.name}'

# Get specific field
k get pod nginx -o jsonpath='{.status.podIP}'

# Multiple fields
k get pod nginx -o jsonpath='{.metadata.name} {.status.podIP}'
```

## With Newlines
```bash
# Range loop with newline
k get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'

# Multiple fields per line
k get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'
```

## Filtering
```bash
# Filter by condition
k get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Filter pods by status
k get pods -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}'
```

## Sorting
```bash
k get pv --sort-by=.spec.capacity.storage
k get pods --sort-by=.metadata.creationTimestamp
k get events --sort-by=.lastTimestamp
```

---

## Custom Columns
```bash
# Basic
k get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP

# More examples
k get pods -o custom-columns="NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase"

k get pv -o custom-columns="NAME:.metadata.name,CAPACITY:.spec.capacity.storage,STATUS:.status.phase"

k get nodes -o custom-columns="NAME:.metadata.name,CPU:.status.capacity.cpu,MEMORY:.status.capacity.memory"

# With wide output alternative
k get pods -o custom-columns="NAME:.metadata.name,READY:.status.containerStatuses[*].ready,NODE:.spec.nodeName"
```

---

## Common JSONPath Examples

### Pods
```bash
# Pod IPs
k get pods -o jsonpath='{.items[*].status.podIP}'

# Pod names on specific node
k get pods -o jsonpath='{.items[?(@.spec.nodeName=="node01")].metadata.name}'

# Container images
k get pods -o jsonpath='{.items[*].spec.containers[*].image}'
```

### Nodes
```bash
# Internal IPs
k get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Node names
k get nodes -o jsonpath='{.items[*].metadata.name}'

# Taints
k get nodes -o jsonpath='{.items[*].spec.taints[*].key}'
```

### Secrets
```bash
# Decode secret value
k get secret my-secret -o jsonpath='{.data.password}' | base64 -d

# List all keys
k get secret my-secret -o jsonpath='{.data}' | jq 'keys'
```

### Services
```bash
# Service ClusterIPs
k get svc -o jsonpath='{.items[*].spec.clusterIP}'

# NodePorts
k get svc -o jsonpath='{.items[?(@.spec.type=="NodePort")].spec.ports[*].nodePort}'
```

---

## Other Output Formats
```bash
k get pods -o yaml                     # full YAML
k get pods -o json                     # full JSON
k get pods -o wide                     # extra columns
k get pods -o name                     # just resource names (pod/nginx)
k get pods --no-headers                # without header row
```

---

## Combine with Other Tools
```bash
# With jq
k get pods -o json | jq '.items[].metadata.name'
k get secret my-secret -o json | jq -r '.data.password' | base64 -d

# With grep
k get pods -o wide | grep Running
k get events | grep Warning

# Count
k get pods --no-headers | wc -l
k get pods -l app=nginx --no-headers | wc -l
```
