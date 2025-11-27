# Workloads

## Jobs

### Create
```bash
k create job my-job --image=busybox -- echo "hello"
k create job my-job --image=busybox $do -- echo "hi" > job.yaml
k get jobs
k describe job my-job
k logs job/my-job
```

### Job YAML
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  completions: 3               # run 3 successful pods
  parallelism: 2               # run 2 at a time
  backoffLimit: 4              # max retries before fail
  activeDeadlineSeconds: 100   # timeout
  template:
    spec:
      restartPolicy: Never     # or OnFailure
      containers:
      - name: worker
        image: busybox
        command: ['sh', '-c', 'echo hello && sleep 5']
```

---

## CronJobs

### Create
```bash
k create cronjob my-cron --image=busybox --schedule="*/5 * * * *" -- echo "hi"
k get cronjobs
k describe cronjob my-cron
```

### CronJob YAML
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cron
spec:
  schedule: "*/5 * * * *"      # every 5 minutes
  concurrencyPolicy: Forbid    # Allow, Forbid, Replace
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  startingDeadlineSeconds: 100
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: worker
            image: busybox
            command: ['sh', '-c', 'date; echo hello']
```

### Cron Schedule Format
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday = 0)
│ │ │ │ │
* * * * *

Examples:
0 * * * *      # every hour
0 0 * * *      # daily at midnight
*/15 * * * *   # every 15 minutes
0 9 * * 1      # every Monday at 9am
```

---

## DaemonSets
Runs one pod per node (for logging, monitoring, etc).

### Commands
```bash
k get ds -A                                            # list all daemonsets
k describe ds kube-proxy -n kube-system
```

### DaemonSet YAML
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      tolerations:                     # run on all nodes including master
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      containers:
      - name: agent
        image: fluentd
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

---

## StatefulSets
For stateful applications (databases, etc).

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql                   # headless service name
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:                # auto-create PVCs
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### StatefulSet Pod Names
```bash
# Pods get stable names: <statefulset>-<ordinal>
mysql-0
mysql-1
mysql-2

# Headless service DNS
mysql-0.mysql.default.svc.cluster.local
```

---

## ReplicaSets
Usually managed by Deployments, rarely created directly.

```bash
k get rs
k describe rs my-rs
k scale rs my-rs --replicas=5
```
