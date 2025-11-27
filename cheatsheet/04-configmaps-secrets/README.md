# ConfigMaps & Secrets

## ConfigMaps

### Create
```bash
k create cm my-config --from-literal=key=value
k create cm my-config --from-literal=k1=v1 --from-literal=k2=v2
k create cm my-config --from-file=config.txt
k create cm my-config --from-file=mykey=config.txt     # custom key name
k create cm my-config --from-env-file=.env
k get cm my-config -o yaml                             # view contents
```

### Use as Environment Variable
```yaml
# Single key
env:
- name: MY_VAR
  valueFrom:
    configMapKeyRef:
      name: my-config
      key: mykey

# All keys
envFrom:
- configMapRef:
    name: my-config
```

### Mount as Volume
```yaml
volumes:
- name: config-vol
  configMap:
    name: my-config
containers:
- volumeMounts:
  - name: config-vol
    mountPath: /etc/config
```

---

## Secrets

### Create
```bash
k create secret generic my-secret --from-literal=pass=mypassword
k create secret generic my-secret --from-file=ssh-key=~/.ssh/id_rsa
k create secret docker-registry my-reg \
  --docker-server=docker.io \
  --docker-username=user \
  --docker-password=pass
```

### Decode Secret
```bash
k get secret my-secret -o jsonpath='{.data.pass}' | base64 -d
k get secret my-secret -o json | jq -r '.data.pass' | base64 -d
```

### Use as Environment Variable
```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: pass

# All keys
envFrom:
- secretRef:
    name: my-secret
```

### Mount as Volume
```yaml
volumes:
- name: secret-vol
  secret:
    secretName: my-secret
containers:
- volumeMounts:
  - name: secret-vol
    mountPath: /etc/secrets
    readOnly: true
```

## Secret Types
```bash
generic                    # arbitrary key-value
docker-registry            # docker auth
tls                        # TLS cert + key
```
