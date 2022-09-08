## 什么情形下会考虑CIS与CIS-C的混合部署

* 从CIS向CIS-C迁移的过渡阶段
* 需要继续使用k8s Ingress资源、CIS CRD资源、以及早期的cccl模型下的Configmap资源来进行业务发布，但同时又希望对AS3 Configmap发布方式具有更高发布性能。可使用CIS继续监听Ingress、CRD等资源，对于使用AS3语法的Configmap资源则采用CIS-C来发布
* 在架构设计上，需要同时支持Ingress，CRD，AS3 Configmap这三种资源模式的



## 混合部署需要考虑的问题

* 不同的控制器不应同时监控相同的k8s资源，应采用`--namespace`、`--namespace-label`[启动参数](/Architecture/parameters)对监控的资源进行隔离

* 不同的控制器下发的配置对象应分别对应在F5 BIG-IP的不同partition分区内

* 在Overlay CNI环境下，需要考虑不同控制器写入的静态ARP、FDB条目是否存在冲突：

  * FDB: CIS-C会检测BIG-IP上的关于nodes FDB条目是否存在，因此不会发生问题

  * ARP: 对于pod相关的静态ARP，CIS采用覆盖性写入的方式，即CIS周期性刷新所有ARP条目，需要让CIS和CIS-C所写入的ARP条目位于不同的partition下。如果您使用的是高于`2.9.1-20220831`的版本CIS-C，则CIS-C会将ARP写入到`/cis-c-tenant`partition下，因此不会发生冲突
    
    > 提示1：如果您从低于`2.9.1-20220831`（含）升级到更新版本CIS-C，CIS-C会自动将已写入`/Common`partition下的ARP条目修改到`cis-c-tenant`下，`cis-c-tenant`由CIS-C启动时自动创建(当发现其不存在时)。目前，该partition没有参数化；该partition依旧可用于下发用户配置，但不同于其他partition，在删除配置时，它不会被删除，请**尽量**避开此partition下发用户配置。
    
    > 提示2：对于CIS-C低于或等于`2.9.1-20220831`的版本，您也可以选择修改CIS的默认ARP写入partition来避免冲突问题，例如，将启动设置`--flannel-name=/Common/flannel_vxlan`修改为`--flannel-name=/k8s/flannel_vxlan`，则相应ARP条目会写入到k8s partition下（k8s是一个提前手工建立好的partition）。此操作在生产环境操作需谨慎。 在本文以下的案例中使用的是即是修改CIS的参数方式。

* 非Overlay CNI环境或CIS与CIS-C对接不同BIG-IP场景时，无考虑上述静态ARP问题以及目标partition问题

## 案例1：Hub模式下混合部署

### 结构透视图

在本示例中，CIS控制器监控`f5cis` namespace内的AS3发布资源，让CIS-C监控`cis-c-hub`下的AS3发布资源。结构图如下：

![image-20220907082839282](img/image-20220907082839282.png)

### Configmap Yaml示例

#### f5cis-hub-as3-cm yaml

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: f5cis-hub-as3-cm
  namespace: f5cis
  labels:
    f5type: virtual-server
    as3: "true"
data:
  template: |
    {
        "class": "AS3",
        "declaration": {
            "class": "ADC",
            "schemaVersion": "3.18.0",
            "id": "f5cis-as3-nginxdemo-dfdsf",
            "label": "Sample 1",
            "updateMode": "selective",
            "remark": "Simple HTTP application with RR pool",
            "f5cis_as3_2": {
                "class": "Tenant",
                    "nginxdemo_as3_2": {
                    "class": "Application",
                    "template": "generic",
                    "ingress_nginxdemo1": {
                        "class": "Service_HTTP",
                        "virtualAddresses": [
                            "172.16.100.145"
                        ],
                        "remark":"ingress: nginxdemo1",
                        "virtualPort": 80,
                        "pool": "nginxdemo_pool_as3_2"
                    },
                    "nginxdemo_pool_as3_2": {
                        "class": "Pool",
                        "monitors": [
                            "http"
                        ],
                        "members": [
                            {
                                "servicePort": 80,
                                "serverAddresses": [ ]
                            }
                        ]
                    }
                }
            }
        }
    }
```



#### cis-c-hub-as3  Yaml

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-hub-as3-cm
  namespace: cis-c-hub
  labels:
    f5type: virtual-server
    as3: "true"
data:
  template: |
    {
        "class": "AS3",
        "declaration": {
            "class": "ADC",
            "schemaVersion": "3.18.0",
            "id": "f5cis-as3-nginxdemo-dfdsf",
            "label": "Sample 1",
            "updateMode": "selective",
            "remark": "Simple HTTP application with RR pool",
            "f5cis_c_as3_1": {
                "class": "Tenant",
                    "nginxdemo_as3_1": {
                    "class": "Application",
                    "template": "generic",
                    "ingress_nginxdemo1": {
                        "class": "Service_HTTP",
                        "virtualAddresses": [
                            "172.16.100.149"
                        ],
                        "remark":"ingress: nginxdemo1",
                        "virtualPort": 80,
                        "pool": "nginxdemo_pool_as3_1"
                    },
                    "nginxdemo_pool_as3_1": {
                        "class": "Pool",
                        "monitors": [
                            "http"
                        ],
                        "members": [
                            {
                                "servicePort": 80,
                                "serverAddresses": [ ]
                            }
                        ]
                    }
                }
            }
        }
    }
```



#### F5 BIGIP-IP配置效果

![image-20220907083312314](img/image-20220907083312314.png)



### 控制器启动参数示例

#### CIS控制器启动参数

```shell
      containers:
      - args:
        - --bigip-username=$(BIGIP_USERNAME)
        - --bigip-password=$(BIGIP_PASSWORD)
        - --bigip-url=172.16.20.205
        - --bigip-partition=k8s
        - --verify-interval=5
        - --node-poll-interval=5
        - --pool-member-type=cluster
        - --default-ingress-ip=172.16.100.196
        - --namespace=f5cis
        - --log-as3-response
        - --ingress-class=f5
        - --hubmode=true
        - --disable-teems=true
        - --insecure=true
        - --log-level=INFO
        - --http-listen-address=0.0.0.0:9113
        - --flannel-name=/k8s/flannel_vxlan
```



#### CIS-C控制器启动参数

````shell
      containers:
      - args:
        - --bigip-username=$(BIGIP_USERNAME)
        - --bigip-password=$(BIGIP_PASSWORD)
        - --bigip-url=https://172.16.20.205
        - --log-level=debug
        - --flannel-name=flannel_vxlan
        - --namespace=cis-c-hub
        - --hub-mode=true
        - --ignore-service-port
````





## 非Hub模式混合部署

### 结构透视图

在本示例中，CIS控制器监控`as3-2` namespace内的AS3发布资源，让CIS-C监控`as3-1`下的AS3发布资源。结构图如下：

![image-20220907092405329](img/image-20220907092405329.png)

### Confgimap Yaml示例

#### cis-as3-2-as3-cm 示例

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-as3-2-as3-cm
  namespace: as3-2
  labels:
    f5type: virtual-server
    as3: "true"
data:
  template: |
    {
        "class": "AS3",
        "declaration": {
            "class": "ADC",
            "schemaVersion": "3.18.0",
            "id": "f5cis-as3-nginxdemo-dfdsf",
            "label": "Sample 1",
            "updateMode": "selective",
            "remark": "Simple HTTP application with RR pool",
            "f5cis_as3_2": {
                "class": "Tenant",
                    "nginxdemo_as3_2": {
                    "class": "Application",
                    "template": "generic",
                    "ingress_nginxdemo1": {
                        "class": "Service_HTTP",
                        "virtualAddresses": [
                            "172.16.100.145"
                        ],
                        "remark":"ingress: nginxdemo1",
                        "virtualPort": 80,
                        "pool": "nginxdemo_pool_as3_2"
                    },
                    "nginxdemo_pool_as3_2": {
                        "class": "Pool",
                        "monitors": [
                            "http"
                        ],
                        "members": [
                            {
                                "servicePort": 80,
                                "serverAddresses": [ ]
                            }
                        ]
                    }
                }
            }
        }
    }
```



#### cis-c-as3-1-as3-cm示例

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-as3-1-as3-cm
  namespace: as3-1
  labels:
    f5type: virtual-server
    as3: "true"
data:
  template: |
    {
        "class": "AS3",
        "declaration": {
            "class": "ADC",
            "schemaVersion": "3.18.0",
            "id": "f5cis-as3-nginxdemo-as3-1",
            "label": "Sample 1",
            "updateMode": "selective",
            "remark": "Simple HTTP application with RR pool",
            "f5cis_c_as3_1": {
                "class": "Tenant",
                    "nginxdemo_as3_1": {
                    "class": "Application",
                    "template": "generic",
                    "ingress_nginxdemo1": {
                        "class": "Service_HTTP",
                        "virtualAddresses": [
                            "172.16.100.149"
                        ],
                        "remark":"ingress: nginxdemo1",
                        "virtualPort": 80,
                        "pool": "nginxdemo_pool_as3_1"
                    },
                    "nginxdemo_pool_as3_1": {
                        "class": "Pool",
                        "monitors": [
                            "http"
                        ],
                        "members": [
                            {
                                "servicePort": 80,
                                "serverAddresses": [ ]
                            }
                        ]
                    }
                }
            }
        }
    }
```



#### F5 BIGIP-IP配置效果:

相对于以上Hub模式下的配置，Configmap里对应的F5 partiton（tenant）并未发生变化，对象命名也均未发生变化，因此在BIG-IP的最终效果与上图一致。



### 控制器启动参数示例

#### CIS控制器启动参数

```shell
      containers:
      - args:
        - --bigip-username=$(BIGIP_USERNAME)
        - --bigip-password=$(BIGIP_PASSWORD)
        - --bigip-url=172.16.20.205
        - --bigip-partition=k8s
        - --verify-interval=5
        - --node-poll-interval=5
        - --pool-member-type=cluster
        - --default-ingress-ip=172.16.100.196
        - --namespace=as3-2 <<<<< as3-2 namespace
        - --hubmode=false <<<<<<非hub模式
        - --log-as3-response
        - --ingress-class=f5
        - --disable-teems=true
        - --insecure=true
        - --log-level=INFO
        - --http-listen-address=0.0.0.0:9113
        - --flannel-name=/k8s/flannel_vxlan
```



#### CIS-C控制器启动参数

```shell
      containers:
      - args:
        - --bigip-username=$(BIGIP_USERNAME)
        - --bigip-password=$(BIGIP_PASSWORD)
        - --bigip-url=https://172.16.20.205
        - --log-level=debug
        - --flannel-name=flannel_vxlan
        - --namespace=as3-1 <<<<< as3-1 namespace
        - --hub-mode=false  <<<<< 非hub模式
        - --ignore-service-port
```

