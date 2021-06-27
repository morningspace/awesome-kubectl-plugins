# Demo: KubeMacro

This demo will show you some interesting macros that are available on KubeMacro Hub.

Before dive into each macro, to learn all macros installed on your local machine:
```shell
kubectl macro
```

For each macro, to learn more on it, you can use `-h` option to print its help information:
```shell
kubectl macro get-pod-by-svc -h
```

## Macro: get-pod-by-svc

This macro can be used to get all pods associated with a specified service. This is useful when you want to inspect the pods associated with a service.

Without the macro, you will have to print the service definition and look for `spec.selector` to know which pods are associated with this service. For example:
```shell
kubectl get svc kube-dns -n kube-system -o yaml
```

The pods should have label `k8s-app` with value `kube-dns`. Now you can list all pods with this label:
```shell
kubectl get pod -l k8s-app=kube-dns -n kube-system
```

To achieve this by a single command using the macro:
```shell
kubectl macro get-pod-by-svc kube-dns -n kube-system
```

You can also sepcify the output format by using `-o` option:
```shell
kubectl macro get-pod-by-svc kube-dns -n kube-system -o name
kubectl macro get-pod-by-svc kube-dns -n kube-system -o wide
kubectl macro get-pod-by-svc kube-dns -n kube-system -o jsonpath='{.items[*].metadata.name}'
```

You can even pipe the output of this macro with `xargs` for more complicated use case:
```shell
kubectl macro get-pod-by-svc kube-dns -n kube-system -o name | xargs -t kubectl describe -n kube-system
```

## Macro: get-by-owner-ref

This macro can be used to get resource in a namespace along with its ancestors by navigating the resource `metadata.ownerReferences` field. The idea is a bit similar to [kubectl-tree](https://github.com/ahmetb/kubectl-tree).

Using this macro, when you have one resource object, you can reveal its parents and ancestors by walking through the owner references of each object on the chain. The result will be presented as a tree. For example, let's list all core dns pods:
```shell
kubectl get pods -l k8s-app=kube-dns -o name -n kube-system
```

Choose one of them and store into $POD_NAME.
<!--shell
var::input "Please input the pod name" POD_NAME
-->

Then use this macro to list all its parents and ancestors:
```shell
kubectl macro get-by-owner-ref $POD_NAME -n kube-system
```

Because `kubectl get` is used in this macro, you can also specify some of the options supported by `kubectl get` to customize the results.
```shell
kubectl macro get-by-owner-ref pod -l 'k8s-app=kube-dns' -n kube-system
```

You can even list all objects for a certain resource along with their ancestors in a namespace:
```shell
kubectl macro get-by-owner-ref pod -n kube-system
```

## Macro: get-apires

This macro can be used to get API resources in a namespace. This is useful when you want to know what API resources are included in a namespace.

For example, to get all resources in kube-system namespace:
```
kubectl macro get-apires -n kube-system
```

When looks for resources, by default the macro will list all resources in a namespace. You can specify `--include` and `--exclude` option with regular expression to filter the resources to be listed. For example, to only list pods resource and all resources whose names start with deployments:
```shell
kubectl macro get-apires --include '^pods$|^deployments.*$' -n kube-system
```

Or course we can use `-o` to specify the output format:
```shell
kubectl macro get-apires --include '^pods$|^deployments.*$' -o name -n kube-system
```

## Macro: get-apires-name-conflict

This macro is designed to get API resources that belong to different API groups but have the same name. This is useful if you want to check any API resource naming conflict in your cluster.
```shell
kubectl macro get-apires-name-conflict
```