# Instructions

Apply this manifest.

```shell
kubectl apply -k github.com/conoa/k8s-interview/deployment
kubectl get deployments
# NAME    READY   UP-TO-DATE   AVAILABLE   AGE
# pause   1/1     1            1           17s
```

Now delete the `pause` deployment.


# Proposed fix:


I like to have a namespace to work in, not necessary in this case, but doing it :smile: Besides, default namespace is not in kyverno policy!

`kubectl create namespace conoa`

Applying deployment:

`kubectl apply -k github.com/conoa/k8s-interview/deployment -n conoa`

Outputs:

```bash
deployment.apps/pause created
```

Testing first: `kubectl get deployments -n conoa

Output:

```bash
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
pause   1/1     1            1           87s
```

Testing logs of the deployment, just to test: `kubectl logs deployment/pause -n conoa`

Output: ***blank ...***

Deleting deployment as task says:

`kubectl delete deployment -n conoa pause`

No successful feedback.. 

Watches the spec :

`kubectl get deployments pause -n conoa -o yaml  | yq | grep finalizers -I3 `

output:

```bash
  "metadata": {
    "annotations": {
      "deployment.kubernetes.io/revision": "1",
      "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"apps/v1\",\"kind\":\"Deployment\",\"metadata\":{\"annotations\":{},\"finalizers\":[\"k8s.extension.conoa.se/v1beta1\"],\"labels\":{\"app\":\"pause\"},\"name\":\"pause\",\"namespace\":\"conoa\"},\"spec\":{\"replicas\":1,\"selector\":{\"matchLabels\":{\"app\":\"pause\"}},\"template\":{\"metadata\":{\"labels\":{\"app\":\"pause\"}},\"spec\":{\"automountServiceAccountToken\":false,\"containers\":[{\"image\":\"public.ecr.aws/eks-distro/kubernetes/pause:3.9\",\"imagePullPolicy\":\"IfNotPresent\",\"name\":\"pause\",\"resources\":{\"limits\":{\"cpu\":\"10m\",\"memory\":\"10Mi\"},\"requests\":{\"cpu\":\"10m\",\"memory\":\"10Mi\"}},\"securityContext\":{\"allowPrivilegeEscalation\":false,\"capabilities\":{\"drop\":[\"ALL\"]},\"readOnlyRootFilesystem\":true}}],\"restartPolicy\":\"Always\",\"securityContext\":{\"runAsGroup\":65534,\"runAsNonRoot\":true,\"runAsUser\":65534,\"seccompProfile\":{\"type\":\"RuntimeDefault\"}}}}}}\n"
    },
    "creationTimestamp": "2026-11-04T22:01:29Z",
    "deletionGracePeriodSeconds": 0,
    "deletionTimestamp": "2026-11-04T22:04:38Z",
    "finalizers": [
      "k8s.extension.conoa.se/v1beta1"
    ],
    "generation": 2,
```

See that there are finalizers, or just one.

First trying to to see if there are CRDS matching the finalizer

`kubectl api-resources` or `kubectl api-resources | grep k8s.extension.conoa.se`

No CRDS matching, then deleting the finalizer:

`kubectl -n conoa patch deploy pause --type=merge -p '{"metadata":{"finalizers":null}}'`

or using `kubectl edit deployments pause -n conoa` with our favorite editor