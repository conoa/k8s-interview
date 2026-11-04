# Hello01

Apply this manifest to a k8s cluster and get the content of `/hello.txt` from the running pod.

```shell
kubectl apply -k github.com/conoa/k8s-interview/hello01
kubectl get pods
```

## Proposed fix

### Setup

```bash
kubectl apply -k github.com/conoa/k8s-interview/hello01 -n conoa
deployment.apps/hello01 created
```


```bash
kubectl get pods -n conoa
NAME                       READY   STATUS    RESTARTS   AGE
hello01-7d68cbcdf8-gr5q4   1/1     Running   0          44s
```

## Testing cat with kubectl exec
`kubectl exec deployments/hello01 -n conoa -- cat /hello.txt`

```bash
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "0fad958e08be70990b27f49c8c89ffd6b8e5a82ac1180aa5082f7bcbb4f2e4d2": OCI runtime exec failed: exec failed: unable to start container process: exec: "cat": executable file not found in $PATH
```

I guess that would be to easy...


## Testing to get shell

`kubectl -n conoa exec deployments/hello01 -i -t -- sh`

so this is perhaps a distroless container

## Testing with kubect debug

`kubectl -n conoa debug -it hello01-7d68cbcdf8-gr5q4 --target=hello01 --image buzybox`

```bash
Targeting container "hello01". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-9t4z5.
Error from server (Forbidden): pods "hello01-7d68cbcdf8-gr5q4" is forbidden: violates PodSecurity "baseline:latest": non-default capabilities (container "debugger-9t4z5" must not include "SYS_PTRACE" in securityContext.capabilities.add)
```

## removing the talos default restrictions by labling

`kubectl label namespace conoa  pod-security.kubernetes.io/enforce=privileged \
  pod-security.kubernetes.io/warn=privileged \
  pod-security.kubernetes.io/audit=privileged \
  --overwrite`

## retrying with kubectl debug

`kubectl -n conoa debug -it hello01-7d68cbcdf8-gr5q4 --target=hello01 --image buzybox`

output:

```bash
why does this fail:

rogerpc :: github/roger/k8slocalcli ‹main*› » k label namespace conoa  pod-security.kubernetes.io/enforce=privileged \
  pod-security.kubernetes.io/warn=privileged \
  pod-security.kubernetes.io/audit=privileged \
  --overwrite
namespace/conoa labeled
rogerpc :: github/roger/k8slocalcli ‹main*› » kubectl -n conoa debug -it hello01-7d68cbcdf8-gr5q4 --target=hello01 --image buzybox
Targeting container "hello01". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-7s5m2.
Warning: container debugger-7s5m2: Back-off pulling image "buzybox": ErrImagePull: failed to pull and unpack image "docker.io/library/buzybox:latest": failed to resolve reference "docker.io/library/buzybox:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
Warning: container debugger-7s5m2: Back-off pulling image "buzybox": ErrImagePull: failed to pull and unpack image "docker.io/library/buzybox:latest": failed to resolve reference "docker.io/library/buzybox:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
Warning: container debugger-7s5m2: Back-off pulling image "buzybox": ErrImagePull: failed to pull and unpack image "docker.io/library/buzybox:latest": failed to resolve reference "docker.io/library/buzybox:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

## retrying without the typo buzy - > busy

`kubectl -n conoa debug -it hello01-7d68cbcdf8-gr5q4 --target=hello01 --image busybox`

output

```bash
Targeting container "hello01". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-hrtlt.
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # 

```

## I'm in with out typo :see_no_evil: :tada:

using top to see the PID for the app

```bash
Mem: 4667496K used, 61170180K free, 4032K shrd, 123420K buff, 2343628K c
CPU:  0.4% usr  0.3% sys  0.0% nic 98.8% idle  0.0% io  0.0% irq  0.0% s
Load average: 0.07 0.13 0.20 1/1308 57
  PID  PPID USER     STAT   RSS %RSS CPU %CPU COMMAND
    1     0 root     S     5356  0.0  29  0.0 /pause
   57    50 root     R     2464  0.0  11  0.0 top
   50     0 root     S     2448  0.0  14  0.0 sh
/ # cat /proc/1/
arch_status        ksm_stat           sched
attr/              limits             schedstat
autogroup          loginuid           sessionid
auxv               map_files/         setgroups
cgroup             maps               smaps
clear_refs         mem                smaps_rollup
cmdline            mountinfo          stack
comm               mounts             stat
coredump_filter    mountstats         statm
cpuset             net/               status
cwd/               ns/                syscall
environ            oom_adj            task/
exe                oom_score          timens_offsets
fd/                oom_score_adj      timers
fdinfo/            pagemap            timerslack_ns
gid_map            personality        uid_map
io                 projid_map         wchan
ksm_merging_pages  root/
/ # cat /proc/1/hello.txt
cat: can't open '/proc/1/hello.txt': No such file or directory
/ # cat /proc/1/root/hello.txt 
Congratulations, you are at least a junior!
/ # exit
```
