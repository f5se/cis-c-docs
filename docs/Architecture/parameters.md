# 启动参数

* `bigip-url: 必填, Big-IP 的 URL地址。 如: https://10.10.10.10:8443 `
* `bigip-username: 必填, Big-IP 用户的账号名。`
* `bigip-password: 必填, Big-IP 相应账号的密码。`
* `log-level: 可选。输出日志的级别。`
* `flannel-name: 可选。默认值为空。若不填flannel-name，则默认该k8s环境使用calico，填上flannel-name则认为该k8s环境使用flannel插件。`
* `namespace: 标识CIS-C将监听的namespace名称。可以填入多行--namespace，代表CIS-C将监听多个namespace内的资源变化。 `
* `namespace-label：标识CIS-C将监听该类label对应的namespaces。该参数可与namespace参数搭配使用。`
