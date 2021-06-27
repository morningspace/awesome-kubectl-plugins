# Demo: KubeAssert

This demo will work you through the major features that KubeAssert provides.

## Assertions basics

To learn all assertions supported by KubeAssert:
```shell
kubectl assert
```

Firstly, let's see how to use `exist` to assert pods with specified label should exist:
```shell
kubectl assert exist pods -l k8s-app=kube-dns -n kube-system
```

To assert these pods are running:
```shell
kubectl assert exist pods -l k8s-app=kube-dns --field-selector status.phase=Running -n kube-system
```

To verify the above assertions, let's run `kubectl get pods`:
```shell
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

To assert these pods do not in error status, let's use `not-exist`:
```shell
kubectl assert not-exist pods -l k8s-app=kube-dns --field-selector status.phase=Error -n kube-system
```

Next, let's use `num` to assert the pod number in specified namespace should be equal to a certain value:
```shell
kubectl assert num pods -n kube-system -eq 8
```

To verify this, let's see how many pods are in kube-system namespace:
```shell
kubectl get pods -n kube-system
```

We can also use `pod-ready` to assert pods are ready:
```shell
kubectl assert pod-ready --all-namespaces
```

This is not just to check if the pod status is `Ready`. It also checks other status such as `Completed` and count the number of containers that are really ready.

To verify this, let's run `kubectl get pods`:
```shell
kubectl get pods --all-namespaces
```

To assert pods restarts meet specified criteria, we can use `pod-restarts`:
```shell
kubectl assert pod-restarts --all-namespaces -lt 10
```

## Using enhanced exist and exist-not

To use field selector is great, but the default field selector provided by kubectl has some limitation due to it's implemented at server side. For example, you will see below assertion failed:
```shell
kubectl assert exist deployments -l k8s-app=kube-dns --field-selector status.replicas=2 -n kube-system
```

Use `exist-enhanced` will resolve the above issue, because it does filtering at client side by the assertion itself.
```shell
kubectl assert exist-enhanced deployments -l k8s-app=kube-dns --field-selector status.replicas=2 -n kube-system
```

The `exist-enhanced` also supports regex, which is not supported by the default field selector. For example, to assert service accounts existed where the secret name should contain text `controller`:
```shell
kubectl assert exist-enhanced serviceaccounts --field-selector 'secrets[*].name=~controller' --all-namespaces
```

To assert deployments in all namespaces at least have one condition under status where the value of type should be `Available`:
```shell
kubectl assert exist-enhanced deployments --field-selector 'status.conditions[*].type=~Available' --all-namespaces
```

To assert secrets whose name should contain any of the text strings specified as below:
```shell
kubectl assert exist-enhanced secrets --field-selector 'metadata.name=~pod|job|deployment' --all-namespaces
```
