## 迁移背景

用户的业务被组织为AS3 Declaration的内容以ConfigMap的形式部署在K8S集群中。K8S集群资源下发控制器[CIS](https://clouddocs.f5.com/containers/latest/)和CIS-C通过监控资源部署事件([List-Watch](https://github.com/topics/list-watch))，对事件请求做出处理。

CIS和CIS-C对ConfigMap中AS3 Declaration的处理逻辑有所不同：

* CIS依赖BIG-IP上预先安装的AS3 RPM包，实现业务在BIG-IP侧的解析、全量部署，
* CIS-C采用独立的[AS3解析程序](https://gitlab.f5net.com/cis-c/cis-c-as3-parsing)借助从AS3 RPM包中抽离的解析逻辑[f5-as3-parser](https://hub.docker.com/repository/docker/zongzw/as3-parser/tags)在BIG-IP之外完成AS3 Declaration的解析，然后使用iControl Rest实现增量部署。

这两种实现方式的不同带来了CIS和CIS-C的性能差异。

所谓“迁移”，是将K8S集群资源下发控制器从CIS切换为CIS-C的过程。

切换的目的是使用CIS-C取代CIS，接管K8S集群资源到BIG-IP的下发能力，即，迁移完成后，BIG-IP上已下发业务的变更或新的业务下发均由CIS-C接管。

## 迁移解决的核心问题

BIG-IP上有一种特殊的资源类型ltm/virtual-address，它会被另一种资源类型ltm/virtual引用。virtual-address在AS3 Declaration中可以被显式声明，也可以不显式声明。显式声明时，CIS会使用声明的virtual-address名称下发该资源，不显式声明时，CIS会用virtual-address的IP作为名称创建BIG-IP资源并引用。CIS-C中简化了这一方式，不管显式还是不显式声明virtual-address，始终以virtual-address的IP作为名称创建、引用。

举例来说，以下AS3 Declaration包含了两种声明方式：

  ```json
      {
            "action": "deploy",
            "persist": true,
            "class": "AS3",
            "declaration": {
                  "class": "ADC",
                  ...
                  "my_tenant": {
                        "class": "Tenant",
                        "my_app": {
                        "class": "Application",
                        "template": "generic",
                        "my_virtual_address_10.51.19.60": {  # 显式声明方式
                              "class": "Service_Address",
                              ...
                              "virtualAddress": "10.51.19.60"
                        },
                        ...
                        "my_virtual": {
                              "class": "Service_TCP",
                              ...
                              "virtualAddresses": [  # 显式声明后的引用
                                    {
                                    "use": "my_virtual_address_10.51.19.60"
                                    }

                              ],
                              # 不显式声明方式
                              "virtualAddresses": ["10.51.19.60"]
                              "virtualPort": 80
                        }
                  }
            }
      }
  ```

由CIS下发显式声明的virtual-address到BIG-IP上后的资源列表为：

  ```json
      {
            "my_tenant": {
                  "": {
                        "ltm/virtual-address/my_virtual_address_10.51.19.60": {
                        "address": "10.51.19.60",
                        ...
                        }
                  },
                  "my_app": {
                        ...
                        "ltm/virtual/my_virtual": {
                        ...
                        "destination": "/my_tenant/my_virtual_address_10.51.19.60:80",
                        "name": "my_virtual",
                        "partition": "my_tenant"
                        }
                  }
            }
      }
  ```

其中，`"ltm/virtual-address/my_virtual_address_10.51.19.60"`为virtual-address名称，它带有my_virtual_address_前缀。virtual引用时，通过“destination”引用virtual-address的名称。

CIS下发不显式声明的virtual-address资源列表为：

  ```json
      {
            "my_tenant": {
                  "": {
                        "ltm/virtual-address/10.51.19.60": {
                        ...
                        "address": "10.51.19.60",
                        "name": "10.51.19.60",
                        "partition": "my_tenant"
                        }
                  },
                  "my_app": {
                        "ltm/virtual/my_virtual": {
                        ...
                        "destination": "/my_tenant/10.51.19.60:80",
                        "name": "my_virtual",
                        "partition": "my_tenant",
                        }
                  }
            }
      }
  ```
其中，virtual-address资源名称不带`my_virtual_address_`前缀。

CIS-C实现中对这两种输入采取统一的处理方式，即都用virtual-address的IP作为资源名称。

为了能够让CIS-C在替换CIS后继续接管BIG-IP上现有资源，需要采取相应措施，实现从`"ltm/virtual-address/my_virtual_address_10.51.19.60"`到`"ltm/virtual-address/10.51.19.60"`的“迁移”。

## 迁移工具

为了实现迁移，我们需要用到以下几个工具。

* [f5-tool-grep-resources](https://gitee.com/zongzw/f5-tool-grep-resources): 

  用于收集指定BIG-IP上的既有资源列表，并按照特定格式保存为json格式的文本文件。该格式可以被`f5-tool-deploy-rest`工具解析。具体使用方法见[repo中的使用说明](https://gitee.com/zongzw/f5-tool-grep-resources)。

* update-virtual-address-name_refs.py:

  复制以下脚本内容，保存为文件：`update-virtual-address-name_refs.py`

  ```python
  import json
  import sys
  import os
  import re
  
  data = ""
  if len(sys.argv) != 2:
    print("should be given original json file greped by f5-tool-grep-resources tool")
    sys.exit(1)

  with open(sys.argv[1]) as fr:
    data = fr.read()
  
  # replace virtual-address name of ipv6
  matches = re.findall("\"ltm/virtual-address/((.*)_(([0-9a-f]{4}_){7}[0-9a-f]{4}))\"", data)
  for m in matches:
    print(m)
    ipv6 = m[2].replace("_", ":")
    data = data.replace(m[0], ipv6)
  
  # replace virtual destination ':' -> '.'
  matches = re.findall("((([0-9a-f]{4}:){7}[0-9a-f]{4})(:))", data)
  for m in matches:
    print(m)
    data = data.replace(m[0], m[1]+".")
  
  # replace virtual-address name of ipv4
  matches = re.findall("\"ltm/virtual-address/((.*)_(([0-9]{1,3}\.){3}[0-9]{1,3}))\"", data)
  for m in matches:
    print(m)
    data = data.replace(m[0], m[2])
  
  with open(sys.argv[1]+"-updated", "w") as fw:
    fw.write(data)
  ```

* [f5-tool-deploy-rest](https://gitee.com/zongzw/f5-tool-deploy-rest):

  用于部署指定的资源列表，该列表具有特定的格式，由`f5-tool-grep-resources`工具产生。
  
  部署过程中使用iControl Rest + Transaction。
  
  具体使用方法见 [repo中的使用说明](https://gitee.com/zongzw/f5-tool-deploy-rest)。

## 迁移步骤

**本迁移步骤为一次性操作，用于修改CIS下发的既有virtual-address资源的命名方式。**

迁移总体步骤共六步:

1. 停止CIS程序

   确保BIG-IP上现有业务不再有新的变更。

2. 通过UCS备份BIG-IP现有业务配置

   以便在切换CIS-C异常时及时恢复原有配置以切换回CIS。

3. 运行`f5-tool-grep-resources`程序

   生成既有资源列表，保存入文本文件中。

4. 运行`update-virtual-address-name_refs.py`脚本

   修改virtual-address资源名称和引用名称，另存为新的文件，供`f5-tool-deploy-rest`使用。

5. 运行`f5-tool-deploy-rest`程序

   下发修改后的资源列表，将BIG-IP上原有virtual-address及virtual资源更新成CIS-C支持的命名方式。

6. 部署并启动CIS-C程序，测试新旧业务下发

### 具体步骤详解

#### 1. 停止CIS程序

CIS程序通常部署为K8S中的kube-system命名空间中的Deployment。

想要停止CIS程序，我们仅需要执行：

```shell
$ kubectl scale -n kube-system deployment/k8s-bigip-ctlr --replicas=0 
```

为了必要时候我们可以方便回退，暂时不推荐通过`kubectl delete -n kube-system deployment/k8s-bigip-ctlr`的方式停止CIS。

#### 2. 通过UCS备份BIG-IP现有业务配置

* 通过BIG-IP WebUI执行UCS：

  ```shell
    <switch to Common Partition> 
        -> System 
        -> Archives 
        -> Create 
        -> File Name -> 任意名称
        -> Finished
  ```

* 通过BIG-IP TMSH执行UCS：

  ```shell
  $ tmsh save sys ucs my-backup
  Saving active configuration...
  /var/local/ucs/my-backup.ucs is saved.
  ```

#### 3. 运行`f5-tool-grep-resources`程序

`f5-tool-grep-resources` 用于生成既有资源的文件列表，执行方式为：

```shell
$ ./f5-tool-grep-resources-darwin-20230715 --root-kind ltm/virtual-address --root-kind ltm/virtual --output existing-vs-va.json
2023/07/24 18:56:00.279411  [INFO] [-] Initializing BIG-IP: https://10.250.15.109
2023/07/24 18:56:00.667853  [INFO] [-] Ignore 'cis-c-tenant' partition grabbing, use --ignore-common=false to grab 'cis-c-tenant'
2023/07/24 18:56:00.667873  [INFO] [-] Ignore 'Common' partition grabbing, use --ignore-common=false to grab 'Common'
2023/07/24 18:56:00.667921  [INFO] [-] Running with options:
2023/07/24 18:56:00.667927  [INFO] [-]     bigip-password : xxxx
2023/07/24 18:56:00.667937  [INFO] [-]         root-kinds : [ltm/virtual-address ltm/virtual]
2023/07/24 18:56:00.667941  [INFO] [-]          overwrite : true
2023/07/24 18:56:00.667943  [INFO] [-]          log-level : debug
2023/07/24 18:56:00.667946  [INFO] [-]      ignore-common : true
2023/07/24 18:56:00.667950  [INFO] [-]          bigip-url : https://10.250.15.109
2023/07/24 18:56:00.667953  [INFO] [-]     bigip-username : admin
2023/07/24 18:56:00.667957  [INFO] [-]         partitions : [my_tenant]
2023/07/24 18:56:00.667961  [INFO] [-]         skip-kinds : [sys/config sys/folder ltm/snat-translation]
2023/07/24 18:56:00.667964  [INFO] [-]             output : all.json
2023/07/24 18:56:00.667974  [INFO] [-] Starting to grab resources: [ltm/virtual-address ltm/virtual] for partitions:
2023/07/24 18:56:00.667979  [INFO] [-] my_tenant
2023/07/24 18:56:00.667986 [DEBUG] [-] kind: ltm/virtual-address
2023/07/24 18:56:00.759813  [INFO] [-]  -> res : ltm/virtual-address my_tenant my_virtual_address_10.51.19.60
2023/07/24 18:56:00.759860 [DEBUG] [-] kind: ltm/virtual
2023/07/24 18:56:00.839917  [INFO] [-]  -> res : ltm/virtual my_tenant my_virtual
2023/07/24 18:56:00.839956  [INFO] [-] Writing grabbed to existing-vs-va.json
2023/07/24 18:56:00.840618  [INFO] [-] kindlist: [
2023/07/24 18:56:00.840630  [INFO] [-]   "ltm/virtual",
2023/07/24 18:56:00.840633  [INFO] [-]   "ltm/virtual-address"
2023/07/24 18:56:00.840636  [INFO] [-] ]
```

以上命令执行的作用是将BIG-IP上既有ltm/virtual和ltm/virtual-address资源配置收集起来，以特定格式存入文件中。

生成的文件为特定格式的JSON文件。

```json
{
    "my_tenant": {
        "": {
            "ltm/virtual-address/my_virtual_address_10.51.19.60": {
                "address": "10.51.19.60",
                "name": "my_virtual_address_10.51.19.60",
                ...
            }
        },
        "my_app": {
            "ltm/virtual/my_virtual": {
                "destination": "/my_tenant/my_virtual_address_10.51.19.60:80",
                "name": "my_virtual",
                "partition": "my_tenant",
                ...
            }
        }
    }
}
```

其中,
* `"my_tenant"`为partition名称。
* `""`和`"my_app"`为subPath。
* `"ltm/virtual/my_virtual"`为资源类型及名称。

#### 4. 运行`update-virtual-address-name_refs.py`脚本

执行该脚本是为了修改第三步中生成的资源列表文本，将不符合CIS-C命名规则的部分修改掉。

从[迁移工具](#迁移工具)部分脚本的实现可以看出它将所有的virtual-address名称及对它的引用修改为以IP命名的方式。

运行方法为：

```shell
$ python update-virtual-address-name_refs.py existing-vs-va.json
```

* 运行结束后生成 existing-vs-va.json-updated 文件。
* 脚本支持IPv6类型的virtual-address.

修改完成后，新生成的`existing-vs-va.json-updated`内容如下：

```json
{
    "my_tenant": {
        "": {
            "ltm/virtual-address/10.51.19.60": {
                "address": "10.51.19.60",
                "name": "10.51.19.60",
                ...
            }
        },
        "my_app": {
            "ltm/virtual/my_virtual": {
                "destination": "/my_tenant/10.51.19.60:80",
                "name": "my_virtual",
                "partition": "my_tenant",
                ...
            }
        }
    }
}
```

#### 5. 运行`f5-tool-deploy-rest`程序

运行方式：

```shell
$ ./f5-tool-deploy-rest-darwin-20230707 --original existing-vs-va.json --target existing-vs-va.json-updated --single-transaction
2023/07/24 19:16:42.141700  [INFO] [-] Running with options:
2023/07/24 19:16:42.141846  [INFO] [-]          log-level : debug
2023/07/24 19:16:42.141851  [INFO] [-] single-transaction : true
2023/07/24 19:16:42.141853  [INFO] [-]          bigip-url : https://10.250.15.109
2023/07/24 19:16:42.141856  [INFO] [-]     bigip-username : admin
2023/07/24 19:16:42.141858  [INFO] [-]     bigip-password : xxxx
2023/07/24 19:16:42.141860  [INFO] [-]           original : existing-vs-va.json
2023/07/24 19:16:42.141863  [INFO] [-]             target : existing-vs-va.json-updated
2023/07/24 19:16:42.141865  [INFO] [-] Initializing BIG-IP: https://10.250.15.109
2023/07/24 19:16:42.582141 [DEBUG] [-] partitions: Create: [], Delete: [], Updated: [kubernetes my_tenant]
2023/07/24 19:16:42.788243 [DEBUG] [-] generating 'delete' cmds for partition kubernetes's config
2023/07/24 19:16:42.788262 [DEBUG] [-] generating 'deploy' cmds for partition kubernetes's config
2023/07/24 19:16:42.788498 [DEBUG] [-] commands: []
2023/07/24 19:16:43.231893 [DEBUG] [-] generating 'delete' cmds for partition my_tenant's config
2023/07/24 19:16:43.231934 [DEBUG] [-] generating 'deploy' cmds for partition my_tenant's config
2023/07/24 19:16:43.233282 [DEBUG] [-] commands: [{"ResName":"my_virtual","Partition":"my_tenant","Subfolder":"my_app","Kind":"ltm/virtual","Method":"DELETE",..."creationTime":"2023-07-24T10:55:45Z","description":"cbnet-pvt1","destination":"/my_tenant/my_virtual_address_10.51.19.60:80","enabled":true,"evictionProtected":"disabled","fullPath":"/my_tenant/my_app/my_virtual","generation":8621,"gtmScore":0,"ipProtocol":"tcp","kind":"tm:ltm:virtual:virtualstate","lastModifiedTime":"2023-07-24T10:55:45Z","mask":"255.255.255.255","mirror":"disabled","mobileAppTunnel":"disabled","name":"my_virtual","nat64":"disabled","partition":"my_tenant","persist":...":"none",...":true,"ScheduleIt":""}]
2023/07/24 19:16:43.311872 [DEBUG] [-] #### DELETE /tm/ltm/virtual/~my_tenant~my_app~my_virtual
2023/07/24 19:16:43.311894 [DEBUG] [-] DELETE https://10.250.15.109/mgmt/tm/ltm/virtual/~my_tenant~my_app~my_virtual
2023/07/24 19:16:43.311900 [DEBUG] [-] X-F5-REST-Coordination-Id: 1690197402310652
2023/07/24 19:16:43.311904 [DEBUG] [-] Authorization: Basic xxxxxxbase64xxx=
2023/07/24 19:16:43.311908 [DEBUG] [-] Content-Type: application/json
2023/07/24 19:16:43.311911 [DEBUG] [-]
2023/07/24 19:16:43.311914 [DEBUG] [-]
2023/07/24 19:16:43.385108 [DEBUG] [-] #### DELETE /tm/ltm/virtual-address/~my_tenant~my_virtual_address_10.51.19.60
2023/07/24 19:16:43.385154 [DEBUG] [-] DELETE https://10.250.15.109/mgmt/tm/ltm/virtual-address/~my_tenant~my_virtual_address_10.51.19.60
2023/07/24 19:16:43.385163 [DEBUG] [-] Authorization: Basic xxxxxxbase64xxx=
2023/07/24 19:16:43.385169 [DEBUG] [-] X-F5-REST-Coordination-Id: 1690197402310652
2023/07/24 19:16:43.385174 [DEBUG] [-] Content-Type: application/json
2023/07/24 19:16:43.385180 [DEBUG] [-]
2023/07/24 19:16:43.385185 [DEBUG] [-]
2023/07/24 19:16:43.456788 [DEBUG] [-] #### POST /tm/ltm/virtual-address
2023/07/24 19:16:43.456810 [DEBUG] [-] POST https://10.250.15.109/mgmt/tm/ltm/virtual-address
2023/07/24 19:16:43.456815 [DEBUG] [-] Content-Type: application/json
2023/07/24 19:16:43.456819 [DEBUG] [-] Authorization: Basic xxxxxxbase64xxx=
2023/07/24 19:16:43.456823 [DEBUG] [-] X-F5-REST-Coordination-Id: 1690197402310652
2023/07/24 19:16:43.456972 [DEBUG] [-]
2023/07/24 19:16:43.456986 [DEBUG] [-]
2023/07/24 19:16:43.531650 [DEBUG] [-] #### POST /tm/ltm/virtual
2023/07/24 19:16:43.531663 [DEBUG] [-] POST https://10.250.15.109/mgmt/tm/ltm/virtual
2023/07/24 19:16:43.531667 [DEBUG] [-] X-F5-REST-Coordination-Id: 1690197402310652
2023/07/24 19:16:43.531669 [DEBUG] [-] Content-Type: application/json
2023/07/24 19:16:43.531672 [DEBUG] [-] Authorization: Basic xxxxxxbase64xxx=
2023/07/24 19:16:43.531682 [DEBUG] [-] {"addressStatus":"yes",..."synCookieStatus":"not-activated","translateAddress":"enabled","translatePort":"enabled","vlansDisabled":true,"vsIndex":7951}
2023/07/24 19:16:43.531764 [DEBUG] [-]
2023/07/24 19:16:43.531767 [DEBUG] [-]
```

以上命令的作用是将`--original`指向的资源列表升级为`--target`指向的资源列表，`f5-tool-deploy-rest`程序会负责识别、比对、编排成iControl Rest请求，以transactiond的形式实现BIG-IP上资源的下发（重命名）。

#### 6. 部署并启动CIS-C程序，测试新旧业务下发

部署和启动CIS-C的过程请参考文档[部署和安装部分](../quick-start/installation.md)部分。

通过用户自己的运管平台下发相关测试业务，维护人员可自行观测日志，并观察BIG-IP上已下发资源情况。
