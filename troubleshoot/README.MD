## OOM with minikube
### Symptom
`$ minikube start`

`$ kubectl get pods`
```
Unable to connect to the server: dial tcp 192.168.49.2:8443: connect: no route to host
```
`$ kubectl get pods --v=2`
```
I0126 22:37:11.520348   13766 shortcut.go:89] Error loading discovery information: Get "https://192.168.49.2:8443/api?timeout=32s": dial tcp 192.168.49.2:8443: connect: no route to host
Unable to connect to the server: dial tcp 192.168.49.2:8443: connect: no route to host
```
### Investigate
#### 1. Assumptions
apiserver is down
#### 2. Check minikube status
`$ minikube status`
```
minikube
type: Control Plane
host: Stopped
kubelet: Stopped
apiserver: Stopped
kubeconfig: Stopped
timeToStop: Nonexistent
```
#### 3. Restart minikube with enabling debug logs
`$ minikube start --alsologtostderr --v=2`
```
[+]ping ok
[+]log ok
[+]etcd ok
[+]poststarthook/start-kube-apiserver-admission-initializer ok
[+]poststarthook/generic-apiserver-start-informers ok
[+]poststarthook/priority-and-fairness-config-consumer ok
[+]poststarthook/priority-and-fairness-filter ok
[+]poststarthook/start-apiextensions-informers ok
[+]poststarthook/start-apiextensions-controllers ok
[+]poststarthook/crd-informer-synced ok
[+]poststarthook/bootstrap-controller ok
[-]poststarthook/rbac/bootstrap-roles failed: reason withheld
[+]poststarthook/scheduling/bootstrap-system-priority-classes ok
[+]poststarthook/priority-and-fairness-config-producer ok
[+]poststarthook/start-cluster-authentication-info-controller ok
[+]poststarthook/aggregator-reload-proxy-client-cert ok
[+]poststarthook/start-kube-aggregator-informers ok
[+]poststarthook/apiservice-registration-controller ok
[+]poststarthook/apiservice-status-available-controller ok
[+]poststarthook/kube-apiserver-autoregistration ok
[+]autoregister-completion ok
[+]poststarthook/apiservice-openapi-controller ok
healthz check failed
- I0126 22:38:42.923819   13869 api_server.go:221] Checking apiserver healthz at https://192.168.49.2:8443/healthz ...
I0126 22:38:42.937527   13869 api_server.go:241] https://192.168.49.2:8443/healthz returned 200:
ok
```
Everything seems to run normally. Let's list all pods again

`$ kubectl get pods`

Ops!
```
Unable to connect to the server: dial tcp 192.168.49.2:8443: connect: no route to host
```
#### 4. Time to check system logs
`$ tail -n300 /var/log/syslog`
```
Jan 26 22:39:00 nguyen-pc kernel: [ 1404.966862] oom-kill:constraint=CONSTRAINT_MEMCG,nodemask=(null),cpuset=9254711a326bf4f69da440675e1f1bbb79c718a3ebb8d1645ac86a8650d06cc3,mems_allowed=0,oom_memcg=/docker/3f5ee78a4dcdfa7cdd1fcec81f5697c186493cca52d5c45c84f0130af2e5fd5b,task_memcg=/docker/3f5ee78a4dcdfa7cdd1fcec81f5697c186493cca52d5c45c84f0130af2e5fd5b/init.scope,task=systemd,pid=14080,uid=0
Jan 26 22:39:00 nguyen-pc kernel: [ 1404.966882] Memory cgroup out of memory: Killed process 14080 (systemd) total-vm:21868kB, anon-rss:3004kB, file-rss:4kB, shmem-rss:0kB, UID:0 pgtables:76kB oom_score_adj:0
```
> It seems that `minkube` start successfully at first then being killed by the OS due to OOM