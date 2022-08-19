# 启动参数

## 程序组成与交付方式

CIS-C程序由两部分组成：

* Golang程序：基于k8s [client-go](https://github.com/kubernetes/client-go)，封装了BIG-IP [iControlRest](https://clouddocs.f5.com/api/icontrol-rest/)。

  交付方式：docker image [f5devcentral/k8s-bigip-ctlr-c](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)。

  启动参数：见[启动参数解析](#启动参数解析)部分阐述。

* NodeJS程序：基于AS3即[F5Networks/f5-appsvcs-extension](https://github.com/F5Networks/f5-appsvcs-extension/releases)，实现过程中抽取了AS3程序的解析逻辑部分，实现为一个服务(webservice)。

  交付方式：docker image [f5devcentral/cis-c-as3-parser](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)。

  启动参数：无

## 启动参数解析

CIS-C安装部署方式，可参见“[下载与安装](../quick-start/installation.md)”描述。这里详细阐述下GoLang程序的启动参数。


* `--as3-service string`

  可选，用于指定NodeJS程序入口，默认为`http://localhost:8081`。

* `--bigip-url string`

  必填，Big-IP的web管理地址。 如: https://10.10.10.10:8443。

* `--bigip-username string`

  可选，Big-IP用户的账号名，默认为`admin`。

* `--bigip-password string`

  必填，Big-IP 相应账号的密码。

* `--flannel-name string`

  可选。默认值为`""`(空字符串)。
  
  若flannel-name值为空字符串，则标识该k8s网络环境使用calico；
  
  若flannel-name不为空字符串，则表示该k8s网络环境为flannel类型。
  
  不同网络环境的BIG-IP配置参见“[准备工作](../quick-start/index.md)”部分描述。

* `--hub-mode`

  可选，`boolean`类型，标识CIS-C是否使用hub模式，默认为`false`。
  
  hub模式的含义及处理细节请参见“[hub模式](../Use-Cases/hub.md)”描述。

* `--ignore-service-port`
  
  可选，`boolean`类型，标识CIS-C对业务端口的绑定行为：当configmap中配置中`servicePort`不存在或者与实际业务不匹配时，是否绑定实际业务第一组端口。默认为`false`。

* `--kube-config string`

  集群外部署模式下，必选。k8s配置文件路径，例如：~/.kube/config

* `--log-level`

  可选。输出日志的级别，取值为： `debug`, `info`, `warn`, `error`, 默认为`info`。

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

### 完整英文参数列表

启动参数的完整英文描述可以通过执行以下命令获取：

```
$ docker run f5devcentral/k8s-bigip-ctlr-c:<version> /f5-kic-linux --help
```

命令的输出为：

```
$ ./f5-kic-linux --help
Usage of ./f5-kic-linux:
  -as3-service string
    	Optional, as3 service for AS3 declaration parsing (default "http://localhost:8081")
  -bigip-password string
    	Required, BIG-IP password for connection, i.e. admin
  -bigip-url string
    	Required, BIG-IP url, i.e. https://enr.yuxn.com:8443
  -bigip-username string
    	Optional, BIG-IP username for connection, i.e. admin (default "admin")
  -flannel-name string
    	Optional, if not default, BigIP Flannel VxLAN Tunnel name, i.e. fl-tunnel
  -hub-mode
    	Optional, specify whether or not to manage ConfigMap resources in hub-mode
  -ignore-service-port
    	Optional, use the 1st targetPort when Pool's servicePort matching fails
  -kube-config string
    	Required if runs as non-inCluster mode, kubernetes configuration i.e. ~/.kube/config
  -log-level string
    	Optional, logging level: debug info warn error (default "info")
  -namespace value
    	Optional, namespace to watch, can be used multiple times, all namespaces would be watched if neither namespaces nor namespace-label is specified
  -namespace-label string
    	Optional, namespace labels to select for watching, all namespaces would be watched if neither namespaces nor namespace-label is specified