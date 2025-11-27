# CKA Exam Cheat Sheet 2025

## Exam Setup (DO THIS FIRST)
```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
```

---

## Pods
```bash
k run nginx --image=nginx                              # create pod
k run nginx --image=nginx $do > pod.yaml               # generate yaml
k run nginx --image=nginx --port=80                    # with container port
k run nginx --image=nginx --command -- sleep 3600      # custom command
k run nginx --image=nginx -l app=web,env=prod          # with labels
k run busybox --image=busybox --rm -it -- sh           # temp debug pod
k delete pod nginx $now                                # force delete
k get pods -o wide                                     # show node + IP
k get pods --show-labels                               # show labels
k get pods -l env=prod,tier=frontend                   # filter by labels
k get pods --selector env=prod                         # same as -l
k get pods -o custom-columns="NAME:.metadata.name,NODE:.spec.nodeName"
```

---

## Deployments
```bash
k create deploy nginx --image=nginx --replicas=3
k create deploy nginx --image=nginx $do > deploy.yaml
k scale deploy nginx --replicas=5
k set image deploy/nginx nginx=nginx:1.19
k rollout status deploy/nginx
k rollout history deploy/nginx
k rollout undo deploy/nginx
k rollout undo deploy/nginx --to-revision=2
k rollout restart deploy/nginx
k autoscale deploy nginx --min=2 --max=10 --cpu-percent=80
```

---

## Services & Networking
```bash
k expose pod nginx --port=80 --name=nginx-svc          # ClusterIP by default
k expose deploy nginx --port=80 --type=NodePort
k create svc clusterip my-svc --tcp=80:80
k create svc nodeport my-svc --tcp=80:80 --node-port=30080
k get svc
k get endpoints
k get netpol
```

---

## ConfigMaps & Secrets
```bash
k create cm my-config --from-literal=key=value
k create cm my-config --from-file=config.txt
k create cm my-config --from-env-file=.env
k create secret generic my-secret --from-literal=pass=123
k create secret generic my-secret --from-file=ssh-key
k get secret my-secret -o jsonpath='{.data.pass}' | base64 -d    # decode secret
k get cm my-config -o yaml
```

---

## Namespaces
```bash
k create ns dev
k get ns
k config set-context --current --namespace=dev         # switch ns
k get pods -A                                          # all namespaces
k get pods -n kube-system
```

---

## Nodes & Scheduling
```bash
k get nodes
k get nodes -o wide
k describe node node01
k cordon node01                                        # mark unschedulable
k uncordon node01                                      # mark schedulable
k drain node01 --ignore-daemonsets --delete-emptydir-data
k taint nodes node01 key=value:NoSchedule              # add taint
k taint nodes node01 key=value:NoSchedule-             # remove taint
k label nodes node01 disktype=ssd                      # add label
k label nodes node01 disktype-                         # remove label
```

---

## Taints & Tolerations (in pod spec)
```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

---

## Node Affinity (in pod spec)
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorMatches:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

---

## Static Pods
```bash
# manifests go in /etc/kubernetes/manifests/
cat /var/lib/kubelet/config.yaml | grep staticPodPath  # find path
```

---

## Multi-container Pods (sidecar, init, ambassador)
```yaml
# initContainers run before main containers, must complete successfully
initContainers:
- name: init
  image: busybox
  command: ['sh', '-c', 'until nc -z mydb 3306; do sleep 2; done']
```
```yaml
# Sidecar: runs alongside main container, shares volumes/network
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  - name: sidecar-logger                               # sidecar container
    image: busybox
    command: ['sh', '-c', 'tail -f /var/log/nginx/access.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  volumes:
  - name: logs
    emptyDir: {}
```
```yaml
# Ambassador: proxy container for external service
spec:
  containers:
  - name: app
    image: myapp
  - name: ambassador                                   # proxy to external DB
    image: ambassador-proxy
    ports:
    - containerPort: 3306
```

---

## Resource Limits
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

---

## Probes
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 5
```

---

## Volumes & PVs
```bash
k get pv
k get pvc
k get storageclass
```
```yaml
# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
# Mount in pod
volumes:
- name: data
  persistentVolumeClaim:
    claimName: my-pvc
volumeMounts:
- name: data
  mountPath: /data
```

---

## RBAC
```bash
k create sa my-sa                                      # service account
k create role pod-reader --verb=get,list --resource=pods
k create rolebinding pod-rb --role=pod-reader --serviceaccount=default:my-sa
k create clusterrole node-reader --verb=get,list --resource=nodes
k create clusterrolebinding node-rb --clusterrole=node-reader --user=admin
k auth can-i get pods --as=system:serviceaccount:default:my-sa
k auth can-i '*' '*'                                   # check admin
```

---

## Logs & Debugging
```bash
k logs pod-name
k logs pod-name -c container-name                      # multi-container
k logs pod-name --previous                             # crashed container
k logs pod-name -f                                     # follow
k exec -it pod-name -- sh                              # shell into pod
k exec pod-name -- cat /etc/resolv.conf
k describe pod pod-name
k get events --sort-by='.lastTimestamp'
k top nodes
k top pods
```

---

## Jobs & CronJobs
```bash
k create job my-job --image=busybox -- echo "hello"
k create cronjob my-cron --image=busybox --schedule="*/5 * * * *" -- echo "hi"
k get jobs
k get cronjobs
```

---

## Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}          # applies to all pods
  policyTypes:
  - Ingress
  - Egress
---
# Allow specific ingress
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - port: 80
```

---

## Ingress
```bash
k create ingress my-ing --rule="host.com/path=svc:80"
k get ingress
```

---

## etcd Backup & Restore
```bash
# Backup
ETCDCTL_API=3 etcdctl snapshot save /tmp/backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Restore
ETCDCTL_API=3 etcdctl snapshot restore /tmp/backup.db --data-dir=/var/lib/etcd-restore
# Then update etcd manifest to use new data-dir
```

---

## Cluster Upgrade (kubeadm)
```bash
# Control plane
apt update && apt-cache madison kubeadm
apt-get install -y kubeadm=1.29.0-00
kubeadm upgrade plan
kubeadm upgrade apply v1.29.0
apt-get install -y kubelet=1.29.0-00 kubectl=1.29.0-00
systemctl daemon-reload && systemctl restart kubelet

# Worker
kubectl drain node01 --ignore-daemonsets
# ssh to worker, upgrade kubeadm, kubelet, kubectl
kubeadm upgrade node
systemctl daemon-reload && systemctl restart kubelet
kubectl uncordon node01
```

---

## Certificates
```bash
kubeadm certs check-expiration
kubeadm certs renew all
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates
```

---

## Useful Paths
```bash
/etc/kubernetes/manifests/          # static pod manifests
/etc/kubernetes/pki/                # cluster certs
/var/lib/kubelet/config.yaml        # kubelet config
/etc/cni/net.d/                     # CNI config
/var/log/pods/                      # pod logs
/var/log/containers/                # container logs
```

---

## Quick YAML Templates
```bash
k run nginx --image=nginx $do                          # pod
k create deploy nginx --image=nginx $do                # deployment
k create svc clusterip my-svc --tcp=80:80 $do          # service
k create job my-job --image=busybox $do -- echo hi     # job
k create cm my-cm --from-literal=k=v $do               # configmap
k create secret generic my-s --from-literal=k=v $do    # secret
k create sa my-sa $do                                  # serviceaccount
k create role my-role --verb=get --resource=pods $do   # role
k create ingress my-ing --rule="h.com/=svc:80" $do     # ingress
```

---

## JSONPath
```bash
k get pods -o jsonpath='{.items[*].metadata.name}'
k get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
k get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
k get pv --sort-by=.spec.capacity.storage
k get pv -o custom-columns="NAME:.metadata.name,CAPACITY:.spec.capacity.storage"
```

---

## Imperative Edits
```bash
k edit pod nginx                                       # opens in vim
k patch deploy nginx -p '{"spec":{"replicas":5}}'
k replace -f pod.yaml
k apply -f pod.yaml
```

---

## Misc
```bash
k api-resources                                        # list all resource types
k api-resources --namespaced=true                      # namespaced resources
k explain pod.spec.containers                          # inline docs
k explain pod --recursive | less                       # full spec
k diff -f manifest.yaml                                # diff before apply
k get all                                              # common resources
```

---

## DaemonSets
```bash
k get ds -A                                            # list all daemonsets
k describe ds kube-proxy -n kube-system
```
```yaml
# DaemonSet runs one pod per node (logs, monitoring, etc)
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-ds
spec:
  selector:
    matchLabels:
      app: my-ds
  template:
    metadata:
      labels:
        app: my-ds
    spec:
      containers:
      - name: agent
        image: busybox
```

---

## Pod Security / Security Context
```yaml
# Pod-level security context
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
        add: ["NET_ADMIN"]
        drop: ["ALL"]
```

---

## Service Account Token Mount
```yaml
# Disable auto-mount of SA token
spec:
  automountServiceAccountToken: false
  serviceAccountName: my-sa
```

---

## Pod Priority & Preemption
```bash
k get priorityclasses
```
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
---
# Use in pod spec
spec:
  priorityClassName: high-priority
```

---

## Horizontal Pod Autoscaler
```bash
k autoscale deploy nginx --min=2 --max=10 --cpu-percent=80
k get hpa
k describe hpa nginx
```

---

## ConfigMap & Secret as Volume
```yaml
# Mount ConfigMap as volume
volumes:
- name: config-vol
  configMap:
    name: my-config
volumeMounts:
- name: config-vol
  mountPath: /etc/config
---
# Mount Secret as volume
volumes:
- name: secret-vol
  secret:
    secretName: my-secret
volumeMounts:
- name: secret-vol
  mountPath: /etc/secrets
  readOnly: true
```

---

## Env from ConfigMap/Secret
```yaml
# Single key
env:
- name: MY_VAR
  valueFrom:
    configMapKeyRef:
      name: my-config
      key: mykey
- name: MY_SECRET
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: password
---
# All keys as env vars
envFrom:
- configMapRef:
    name: my-config
- secretRef:
    name: my-secret
```

---

## hostPath Volume (use sparingly)
```yaml
volumes:
- name: host-vol
  hostPath:
    path: /data
    type: DirectoryOrCreate
```

---

## emptyDir Volume
```yaml
volumes:
- name: cache
  emptyDir: {}                                         # deleted when pod dies
```

---

## DNS & Service Discovery
```bash
# Service DNS format
<service>.<namespace>.svc.cluster.local
# Pod DNS format
<pod-ip-dashed>.<namespace>.pod.cluster.local
# Check DNS
k exec -it busybox -- nslookup kubernetes.default
k exec -it busybox -- cat /etc/resolv.conf
```

---

## Troubleshooting Services
```bash
k get endpoints my-svc                                 # check if pods are registered
k describe svc my-svc                                  # check selector
k get pods -l app=myapp                                # check matching pods
k exec -it test-pod -- curl my-svc:80                  # test connectivity
```

---

## Troubleshooting Nodes
```bash
systemctl status kubelet                               # kubelet status
journalctl -u kubelet -f                               # kubelet logs
systemctl restart kubelet
cat /var/lib/kubelet/config.yaml                       # kubelet config
ls /etc/kubernetes/manifests/                          # control plane pods
crictl ps                                              # container runtime
crictl logs <container-id>
```

---

## Kubectl Context & Config
```bash
k config view
k config get-contexts
k config current-context
k config use-context my-cluster
k config set-context --current --namespace=dev
```

---

## Quick vim tips (for exam)
```bash
:set number                                            # line numbers
:set paste                                             # paste mode (no auto-indent)
/pattern                                               # search
n / N                                                  # next/prev match
dd                                                     # delete line
yy                                                     # copy line
p                                                      # paste
u                                                      # undo
:wq                                                    # save and quit
```

---

## Exam Tips
```bash
# Use kubectl explain liberally
k explain pod.spec.containers.livenessProbe
k explain pv.spec --recursive | grep -A5 capacity

# Copy pod to debug
k get pod my-pod -o yaml > debug.yaml
# Edit and recreate

# Quick pod test
k run test --image=busybox --rm -it -- wget -qO- http://my-svc

# Check cluster info
k cluster-info
k get componentstatuses                                # deprecated but may work

# Count resources
k get pods --no-headers | wc -l
k get pods --no-headers -l app=nginx | wc -l
```

