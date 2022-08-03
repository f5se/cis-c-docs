# 卸载

如果部署时 CIS-C deployment 的名称为 k8s-bigip-ctlr-c，且属于kube-system 命名空间，则可以使用类似如下命令删除 CIS-C deployment。
```
kubectl delete deployments.apps -n kube-system k8s-bigip-ctlr-c
```

* `请谨慎做删除的操作。`
