# Instructions

Apply this manifest.

```shell
kubectl apply -k github.com/conoa/k8s-interview/deployment
kubectl get deployments
# NAME    READY   UP-TO-DATE   AVAILABLE   AGE
# pause   1/1     1            1           17s
```

Now delete the `pause` deployment.
