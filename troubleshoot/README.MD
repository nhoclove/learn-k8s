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
`$ minikube logs -f`
```
[  +0.012319] Memory cgroup out of memory: Killed process 10711 (storage-provisi) total-vm:732832kB, anon-rss:3520kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:148kB oom_score_adj:1000
[ +10.888019] dockerd invoked oom-killer: gfp_mask=0x100cca(GFP_HIGHUSER_MOVABLE), order=0, oom_score_adj=-999
[  +0.000005] CPU: 3 PID: 7724 Comm: dockerd Tainted: G           OE     5.8.0-38-generic #43~20.04.1-Ubuntu
```
> It seems that `minkube` start successfully at first then being killed by the OS due to OOM
#### 5. Check system resource
```
$ free -h

              total        used        free      shared  buff/cache   available
Mem:          7.7Gi       2.0Gi       1.7Gi       424Mi       3.9Gi       5.1Gi
Swap:         2.0Gi       0.0Ki       2.0Gi

$ top

Tasks: 257 total,   1 running, 256 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.0 us,  1.4 sy,  0.0 ni, 95.4 id,  0.1 wa,  0.0 hi,  0.1 si,  0.0 st
MiB Mem :   7872.0 total,   1706.8 free,   2145.4 used,   4019.8 buff/cache
MiB Swap:   2048.0 total,   2047.7 free,      0.2 used.   5156.1 avail Mem
```
From minikube documentation. The system required for minikube as follows:
- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMWare
> My system can fulfil all of these requirements above. Why I have OOM ?

> I just review my cassandra-statefulset and see that for each Pod I request
> 1Gi of memory and 500m of CPU, currently that resource starts 3 replicas
> which may consume up to 3Gi of memory
```
    resources:
        limits:
        cpu: "500m"
        memory: 1Gi
        requests:
        cpu: "500m"
        memory: 1Gi
```
it seems suspicious here
**Let's dig deeper about OOM**
- Suspicion: I set memory limit for each cassandra Container can use up to 1Gi but cassandra Container somehow
might have used more than what It was allocated so OOM was triggered.
If that's the case then only cassandra Container was killed not the whole minikube program.
But according to [this](https://github.com/docker/for-linux/issues/1001) it can happen

**Show system process tree**
`$ pstree -n`
```
├─containerd─┬─16*[{containerd}]
    │        └─containerd-shim─┬─10*[{containerd-shim}]
    │                          └─systemd─┬─systemd-journal
    │                                    ├─containerd───19*[{containerd}]
    │                                    ├─sshd
    │                                    ├─dockerd───50*[{dockerd}]
    │                                    ├─kubelet───19*[{kubelet}]
    │                                    ├─2*[containerd-shim─┬─13*[{containerd-shim}]]
    │                                    │                    └─pause]
    │                                    ├─3*[containerd-shim─┬─13*[{containerd-shim}]]
    │                                    │                    └─pause]
    │                                    ├─containerd-shim─┬─12*[{containerd-shim}]
    │                                    │                 └─etcd───14*[{etcd}]
    │                                    ├─containerd-shim─┬─11*[{containerd-shim}]
    │                                    │                 └─kube-scheduler───8*[{kube-scheduler}]
    │                                    ├─containerd-shim─┬─12*[{containerd-shim}]
    │                                    │                 └─kube-controller───7*[{kube-controller}]
    │                                    ├─containerd-shim─┬─12*[{containerd-shim}]
    │                                    │                 └─kube-apiserver───10*[{kube-apiserver}]
    │                                    ├─containerd-shim─┬─12*[{containerd-shim}]
    │                                    │                 └─pause
    │                                    ├─containerd-shim─┬─14*[{containerd-shim}]
    │                                    │                 └─pause
    │                                    ├─containerd-shim─┬─12*[{containerd-shim}]
    │                                    │                 └─coredns───9*[{coredns}]
    │                                    ├─containerd-shim─┬─12*[{containerd-shim}]
    │                                    │                 └─kube-proxy───7*[{kube-proxy}]
    │                                    └─containerd-shim─┬─12*[{containerd-shim}]
    │                                                      └─storage-provisi───5*[{storage-provisi}]

```
We can see that there is one `containerd` process then under it there is a `systemd` process
then child processes such as kube-apiserver, coredns, kube-scheduler.
### References:
- https://github.com/docker/for-linux/issues/1001
- https://docs.docker.com/config/containers/resource_constraints/#--kernel-memory-details
- https://docs.docker.com/config/containers/resource_constraints/