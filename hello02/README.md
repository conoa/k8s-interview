# Hello02

Apply this manifest to a k8s cluster and get the content of `/hello.txt` from the running pod.

```shell
kubectl apply -k github.com/conoa/k8s-interview/hello02
kubectl get pods
```


## Proposed fix

### setup

```bash
kubectl apply -k github.com/conoa/k8s-interview/hello02 -n conoa        130 ↵
deployment.apps/hello02 created
```

```bash
k get pods -n conoa                                                     
NAME                       READY   STATUS    RESTARTS   AGE
hello01-7d68cbcdf8-gr5q4   1/1     Running   0          31m
hello02-5cd67885c-5qkxc    1/1     Running   0          2m24s

```

## is this the same as hello01 :thinking: without the typo issues ...

jumping to the assumption that this is distroless container:

`kubectl -n conoa debug -it hello02-5cd67885c-5qkxc --target=hello01 --image busybox`

output:

```bash
Targeting container "hello02". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-x24p9.
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # 
```

## ok, I am in 

using ps this time to check the PIDs

```bash
/ # ps
PID   USER     TIME  COMMAND
    1 1337      0:00 /pause
   32 root      0:00 sh
   57 root      0:00 sh
   65 root      0:00 ps
/ # 
```

## cat /proc/1/root/hello.txt 

output:

```bash
/ # cat /proc/1/root/hello.txt 
Congratulations, you are becoming a senior!
```

I am a bit surprised, that this is the same... 

Learning that the kubernetes defaults now to --profile=general for kubectl debug from AI :robot: 