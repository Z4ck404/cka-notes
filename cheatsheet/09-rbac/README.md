# RBAC

## Service Accounts
```bash
k create sa my-sa                                      # create service account
k get sa                                               # list service accounts
k describe sa my-sa
```

### Use in Pod
```yaml
spec:
  serviceAccountName: my-sa
  automountServiceAccountToken: false                  # disable token mount
```

---

## Roles (namespace-scoped)
```bash
k create role pod-reader --verb=get,list,watch --resource=pods
k create role pod-reader --verb=get --resource=pods --resource-name=my-pod
k get roles
k describe role pod-reader
```

### Role YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]                  # "" = core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "delete"]
```

---

## RoleBindings
```bash
k create rolebinding pod-rb --role=pod-reader --user=jane
k create rolebinding pod-rb --role=pod-reader --serviceaccount=default:my-sa
k create rolebinding pod-rb --role=pod-reader --group=developers
k get rolebindings
k describe rolebinding pod-rb
```

### RoleBinding YAML
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-rb
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-sa
  namespace: default
- kind: User
  name: jane
- kind: Group
  name: developers
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## ClusterRoles (cluster-scoped)
```bash
k create clusterrole node-reader --verb=get,list --resource=nodes
k create clusterrole secret-reader --verb=get --resource=secrets
k get clusterroles
```

---

## ClusterRoleBindings
```bash
k create clusterrolebinding node-rb --clusterrole=node-reader --user=admin
k create clusterrolebinding node-rb --clusterrole=node-reader --serviceaccount=default:my-sa
k get clusterrolebindings
```

---

## Check Permissions
```bash
k auth can-i get pods                                  # check self
k auth can-i get pods --as=jane                        # check as user
k auth can-i get pods --as=system:serviceaccount:default:my-sa
k auth can-i '*' '*'                                   # check if admin
k auth can-i --list                                    # list all permissions
k auth can-i --list --as=jane
```

---

## Common Verbs
```
get, list, watch       # read
create, update, patch  # write
delete, deletecollection
```

## Common Resources
```
pods, services, deployments, configmaps, secrets
nodes, namespaces, persistentvolumes (cluster-scoped)
```
