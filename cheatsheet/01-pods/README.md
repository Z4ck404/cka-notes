# Pods

## Pod States
| State | Description |
|-------|-------------|
| **Pending** | Waiting for scheduling or image pull |
| **Running** | At least one container running |
| **Succeeded** | All containers exited with 0 |
| **Failed** | At least one container failed |
| **CrashLoopBackOff** | Container keeps crashing, backing off restart |

## Create Pods
```bash
k run nginx --image=nginx                              # create pod
k run nginx --image=nginx $do > pod.yaml               # generate yaml
k run nginx --image=nginx --command -- sleep 3600      # custom command
k run busybox --image=busybox --rm -it -- sh           # temp debug pod
k delete pod nginx $now                                # force delete
```

## Get Pods
```bash
k get po                                               # list pods
k get po -o wide                                       # show node + IP
k get po --show-labels                                 # show labels
k get po -l app=nginx                                  # filter by label
k get po -o yaml                                       # full yaml
k describe po nginx                                    # detailed info
```

## Edit/Debug
```bash
k edit po nginx                                        # edit live pod
k logs nginx                                           # view logs
k logs nginx -c sidecar                                # specific container
k logs nginx --previous                                # crashed container
k exec -it nginx -- sh                                 # shell into pod
k exec nginx -- cat /etc/hosts                         # run command
```

## Pod YAML (Minimal)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

## With Resources
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "64Mi"
      limits:
        cpu: "500m"
        memory: "128Mi"
```
