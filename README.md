# Conoa Interview tests 

We want know how you tried to solve these tasks. There are multiple ways to solve them.

Good luck!

## Instructions

- You'll need a k8s cluster (local or remote, doesn't matter). If you don't have a local or remote cluster we can recommend [minikube](https://minikube.sigs.k8s.io/).
- You can start with task 1 and 2. Task 3 is optional and similar as task 2 but with a small difference. 
- Now go into each task folder for task specific instructions.

## Tasks

Go into the folders, follow instructions.

1. [deployment](deployment)
1. [hello01](hello01)
1. [hello02](hello02)


# Solution proposal


## Create a local test kubernetes cluster

Using https://github.com/rogerwesterbo/k8slocalcli (or https://github.com/rogerwesterbo/createlocalk8s) as a TUI tool to either use Kind og Talos for talos. I recommend using Talos as a immutable OS. 

Git clone the repo to you local disk, and do: `k8slocalcli create`

gives this:

```bash
create              1 ↵
ℹ️  Removed stale Talos state /home/roger/.k8slocalcli/clusters/conoatest (no containers for "conoatest")

⏰ Creating Talos cluster "conoatest" (k8s 1.37.2, CNI default, 1 control plane, 2 worker(s))
$ talosctl cluster create docker --name conoatest --state /home/roger/.k8slocalcli/clusters --workers 2 --kubernetes-version 1.37.2 --memory-controlplanes 2048MB --memory-workers 2048MB --exposed-ports 80:80/tcp,443:443/tcp
generating PKI and tokens
creating state directory in "/home/roger/.k8slocalcli/clusters/conoatest"
downloading ghcr.io/siderolabs/talos:v1.14.5
creating network conoatest
creating controlplane nodes
creating worker nodes
waiting for Talos API (to bootstrap the cluster)
bootstrapping cluster
waiting for etcd to be healthy: ...
waiting for etcd to be healthy: 1 error occurred:
    * 10.5.0.2: service "etcd" not in expected state "Running": current state [Preparing] Running pre state
waiting for etcd to be healthy: 1 error occurred:
    * 10.5.0.2: service is not healthy: etcd
waiting for etcd to be healthy: OK
waiting for etcd members to be consistent across nodes: ...
waiting for etcd members to be consistent across nodes: OK
waiting for etcd members to be control plane nodes: ...
waiting for etcd members to be control plane nodes: OK
waiting for apid to be ready: ...
waiting for apid to be ready: OK
waiting for all nodes memory sizes: ...
waiting for all nodes memory sizes: OK
waiting for all nodes disk sizes: ...
waiting for all nodes disk sizes: OK
waiting for no diagnostics: ...
waiting for no diagnostics: OK
waiting for kubelet to be healthy: ...
waiting for kubelet to be healthy: OK
waiting for all nodes to finish boot sequence: ...
waiting for all nodes to finish boot sequence: OK
waiting for all k8s nodes to report: ...
waiting for all k8s nodes to report: Get "https://127.0.0.1:44553/api/v1/nodes": EOF
waiting for all k8s nodes to report: can't find expected node with IPs ["10.5.0.2"]
waiting for all k8s nodes to report: OK
waiting for all control plane static pods to be running: ...
waiting for all control plane static pods to be running: OK
waiting for all control plane components to be ready: ...
waiting for all control plane components to be ready: OK
waiting for all k8s nodes to report ready: ...
waiting for all k8s nodes to report ready: OK
waiting for kube-proxy to report ready: ...
waiting for kube-proxy to report ready: OK
waiting for coredns to report ready: ...
waiting for coredns to report ready: OK
waiting for all k8s nodes to report schedulable: ...
waiting for all k8s nodes to report schedulable: OK

merging kubeconfig into "/home/roger/.kube/config"
PROVISIONER           docker
NAME                  conoatest
NETWORK NAME          conoatest
NETWORK CIDR          10.5.0.0/24
NETWORK GATEWAY       10.5.0.1
NETWORK MTU           1500
KUBERNETES ENDPOINT   https://127.0.0.1:44553

NODES:

NAME                        TYPE           IP         CPU    RAM      DISK
/conoatest-controlplane-1   controlplane   10.5.0.2   2.00   2.0 GB   -
/conoatest-worker-1         worker         10.5.0.3   2.00   2.0 GB   -
/conoatest-worker-2         worker         10.5.0.4   2.00   2.0 GB   -

⏰ Merging kubeconfig
$ talosctl kubeconfig --cluster conoatest --nodes 127.0.0.1 --context conoatest --force
🔧 Pointing kubeconfig cluster "conoatest" at https://127.0.0.1:44553
$ kubectl config set-cluster conoatest --server=https://127.0.0.1:44553
Cluster "conoatest" set.
$ kubectl config use-context admin@conoatest
Switched to context "admin@conoatest".

✅ Talos cluster "conoatest" is ready

kubectl context: admin@conoatest
Try: kubectl --context admin@conoatest get nodes
Kubeconfig: k8slocalcli kubeconfig conoatest --output conoatest.kubeconfig
```

Get the Kubeconfig file:

`k8slocalcli kubeconfig conoatest --output conoatest.kubeconfig`

Use the kubeconfig in bash/zsh: `export KUBECONFIG=./conoatest.kubeconfig`

Test kubeconfig: `kubectl get nodes`, gives this:

```bash
rogerpc :: » k get no
NAME                       STATUS   ROLES           AGE   VERSION
conoatest-controlplane-1   Ready    control-plane   83s   v1.37.2
conoatest-worker-1         Ready    <none>          83s   v1.37.2
conoatest-worker-2         Ready    <none>          83s   v1.37.2
```

Yay, we have a working kubernetes cluster with latest version :tada: