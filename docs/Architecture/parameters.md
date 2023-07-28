# 启动参数

## 程序组成与交付方式

CIS-C发布为两个Docker镜像：

* [f5devcentral/k8s-bigip-ctlr-c](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)：基于k8s [client-go](https://github.com/kubernetes/client-go)，封装了BIG-IP [iControlRest](https://clouddocs.f5.com/api/icontrol-rest/)。

* [f5devcentral/cis-c-as3-parser](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)：NodeJS程序，负责[AS3](https://github.com/F5Networks/f5-appsvcs-extension/releases)程序的解析。


### 完整参数列表

启动参数的完整英文描述可以通过执行以下命令获取：

```
$ docker run f5devcentral/k8s-bigip-ctlr-c:<version> /f5-kic-linux --help
```

命令的输出为：

```shell
$ docker run f5devcentral/k8s-bigip-ctlr-c:2.14.6-20230728 /f5-kic-linux --help
Usage of /f5-kic-linux:
  -as3-service string
    	Optional, as3 service for AS3 declaration parsing (default "http://localhost:8081")
  -bigip-password string
    	Required, BIG-IP password for connection, i.e. admin
  -bigip-url string
    	Required, BIG-IP url, i.e. https://enr.yuxn.com:8443
  -bigip-username string
    	Optional, BIG-IP username for connection, i.e. admin (default "admin")
  -credentials-directory string
    	Optional, directory that contains the BIG-IP username, password, and/orurl files. To be used instead of username, password, and/or url arguments.
  -dry-run
    	Optional, run with dry-run mode to skip the startup event handling, used in CIS to CIS-C migration usecase
  -flannel-name string
    	Optional, if not default, BigIP Flannel VxLAN Tunnel name, i.e. fl-tunnel
  -flannel-name-v6 string
    	Optional, default is empty. BigIP Flannel VxLAN Tunnel name for ipv6, i.e. fl-tunnel-v6
  -health-check-interval int
    	Optional, the health check interval, in seconds (default 15)
  -hub-mode
    	Optional, specify whether or not to manage ConfigMap resources in hub-mode
  -ignore-service-port
    	Optional, use the 1st targetPort when Pool's servicePort matching fails
  -kube-config string
    	Required if runs as non-inCluster mode, kubernetes configuration i.e. ~/.kube/config
  -log-level string
    	Optional, logging level: trace debug info warn error (default "info")
  -namespace value
    	Optional, namespace to watch, can be used multiple times, all namespaces would be watched if neither namespaces nor namespace-label is specified
  -namespace-label string
    	Optional, namespace labels to select for watching, all namespaces would be watched if neither namespaces nor namespace-label is specified
  -pool-member-type string
    	Optional, the pool member type if v1.Service's 'type' is 'NodePort', valid values: cluster|nodeport (default "cluster")
  -sys-save-interval int
    	Optional, the interval to run 'tmsh save sys config' for resource persistence, in seconds (default 3600)
```

## 启动参数解析

CIS-C安装部署方式，可参见“[下载与安装](../quick-start/installation.md)”描述。这里主要详细阐述各启动参数含义及使用场景。

* `--as3-service string`

  可选，用于指定NodeJS程序入口，默认为`http://localhost:8081`。也可以指定为BIG-IP管理地址URL。

  如果指定为BIG-IP管理地址URL，需要BIG-IP预先安装AS3 RPM。

* `--bigip-url string`

  可选，Big-IP的web管理地址。 如: https://10.10.10.10:8443。

* `--bigip-username string`

  可选，Big-IP用户的账号名，默认为`admin`。

* `--bigip-password string`

  可选，Big-IP 相应账号的密码。

* `--credentials-directory string`

  可选。该目录里包含三个文件。文件名分别为 "username", "password" 和 "url"，分别对应 BIGIP 的用户名、密码和URL。
  这些文件里的内容比传入的 --bigip-url,--bigip-username 或 --bigip-password 参数优先级更高.
  使用方式参考 “[下载与安装](../quick-start/installation.md)” 。

* `--dry-run`

  可选，默认 false, 以“空转”的方式启动CIS-C，启动后CIS-C接收K8S资源事件后不会将其下发到BIG-IP，只是做转换和记录。

  dry-run的方式主要用于CIS到CIS-C的迁移过程。
  
  具体来说，自停止CIS到启动CIS-C这段时间内，如果确定K8S侧没有新的下发请求事件，则可以通过dry-run模式启动CIS-C，这样CIS-C会忽略程序启动后这段时间内K8S通过informer同步过来的资源事件，转换并记录到data-group（cis-c-tenant下，以f5_kic_*开头的data-group），即表示这些资源已经下发到BIG-IP。

  启动完成后（在日志中观察，不再有新的资源事件到来时），可以通过CIS-C暴露的`API/hook/settings?dry-run=false`停止dry-run模式，CIS-C之后会正常工作，接收（K8S资源事件）、转换（下发请求）、下发（BIG-IP资源）、记录（下发状态）。

  具体使用方法见[动态参数调整](#动态参数调整)部分。

* `--flannel-name string`

  可选。默认值为`""`(空字符串)。

  若flannel-name不为空字符串，则表示该k8s网络环境使用ipv4 flannel vxlan场景.

  不同网络环境的BIG-IP配置参见“[准备工作](../quick-start/index.md)”部分描述。

* `--flannel-name-v6 string`

  可选。默认值为`""`(空字符串)。

  若flannel-name-v6不为空字符串，则表示该k8s网络环境使用ipv6 flannel vxlan场景.
  
  若flannel-name 与 flannel-name-v6 同时设置，则意味着环境为 v4 与 v6 共存环境。
  
  若flannel-name 与 flannel-name-v6 同时为空，则标识该环境使用非flannel网络。
  
  不同网络环境的BIG-IP配置参见“[准备工作](../quick-start/index.md#big-ip侧配置基础网络)”部分描述。

* `--health-check-interval`

  可选，默认15，单位秒，该参数目前仅用于程序调试监控用，通过过滤日志 "pendingDeploys queue length"，可以查看当前请求时间队列的长度，进而判定资源下发的响应能力情况。可动态调整，详见[动态参数调整](#动态参数调整)部分描述。

* `--hub-mode`

  可选，`boolean`类型，标识CIS-C是否使用hub模式，默认为`false`。
  
  hub模式的含义及处理细节请参见“[hub模式](../Use-Cases/hub.md)”描述。

* `--ignore-service-port`
  
  可选，`boolean`类型，标识CIS-C对业务端口的绑定行为：当configmap中配置中`servicePort`不存在或者与实际业务不匹配时，是否绑定实际业务第一组端口。默认为`false`。

* `--kube-config string`

  集群外部署模式下，必选。k8s配置文件路径，例如：~/.kube/config

* `--log-level`

  可选。输出日志的级别，取值为：`trace`, `debug`, `info`, `warn`, `error`, 默认为`info`。可动态调整，详见[动态参数调整](#动态参数调整)部分描述。

* `--namespace string`

  标识CIS-C将监听的namespace名称。
  
  可以多次使用此参数，CIS-C将监听所有指定的namespace内的资源变化。

  该参数可与`--namespace-label`参数混合使用，最终被监控的namespaces 取两者合集。

  如果`--namespace`和`--namespace-label`均未被使用，则所有namespace下的资源都会被监控。

  当与`--hub-mode`参数联用时，`--namespace`和`--namespace-label`标识含义有所不同，参见“[hub模式](../Use-Cases/hub.md)”描述。

* `--namespace-label string`

  标识CIS-C将监听该label筛选后的namespaces。
  
  该参数可与`--namespace`参数混合使用，最终被监控的namespaces 取两者合集。

  如果`--namespace`和`--namespace-label`均未被使用，则所有namespace下的资源都会被监控。

  当与`--hub-mode`参数联用时，`--namespace`和`--namespace-label`标识含义有所不同，参见“[hub模式](../Use-Cases/hub.md)”描述。

* `--pool-member-type`

  可选，默认cluster，当下发的Service资源 serviceType为NodePort时，确认要以何种方式定义BIG-IP上的member。

  当 --pool-member-type cluster时，无论ServiceType为NodePort 还是ClusterIP，均以Pod地址作为member地址。

  当 --pool-member-type nodeport时，NodePort类型的Service会以K8S节点IP作为member地址；ClusterIP类型的Service仍然以Pod地址作为member地址。

  即：

  |pool-member-type\ServiceType|NodePort|ClusterIP|
  |---|---|---|
  |cluster|PodIP|PodIP|
  |nodeport|NodeIP|PodIP

* `--sys-save-interval`

  可选，默认3600，单位秒，周期性执行 `$ tmsh save sys config`的时间间隔。可动态调整，详见[动态参数调整](#动态参数调整)部分描述。

## 动态参数调整

CIS-C支持部分参数动态调整，使用方法为

```shell
$ curl http://<cis-c-url>:8080/hook/settings?参数名=参数值 -X OPTIONS
```

例如：

```shell
$ curl 10.250.16.116:30080/hook/settings?log-level=debug\&health-check-interval=30 -X OPTIONS
{"request_id": "f9121af1-3ed7-431c-811d-46524ef6e1cd","message": "settings are updated:  health-check-interval  log-level "}
```

参数包括：

* `dry-run`

  用于迁移场景中，当集群资源规模较大时的启动加速。具体参见[dry-run参数讲解](#启动参数解析)

  目前仅支持 `dry-run=false`，不支持 `dry-run=true`，即不支持动态进入dry-run模式。

  当被调用`dry-run=false`时，CIS-C会强制刷写运行时配置到data-group中。

* `sys-save-interval`

  动态调整 `tmsh save sys config`的执行间隔。此参数可以适当改大，比如 `sys-save-interval=86400`，但不建议改小，否则CIS-C会持续唤起save sys config，导致系统处理事件请求的响应变慢。

* `health-check-interval`

  动态调整健康检查时间间隔。

* `log-level`

  动态调整日志级别，已经开始的事件处理的日志级别不受影响，从下一个事件处理开始生效。

  此参数可以用于问题的诊断定位，例如 `log-level=trace`可以打开更详细日志，在问题定位结束后，关闭调试：`log-level=info`，或者`log-level=error`达到静默效果。