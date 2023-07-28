# Prometheus集成

Prometheus 是新一代的云原生监控系统，和Kubernetes有着天然的兼容关系，所以在CIS-C项目中，我们首推使用Prometheus，作为监控手段实现CIS-C的性能评估和运行状态收集。

## 服务器端搭建

Prometheus 的搭建方式有很多种，本文档中展示的是以docker方式搭建独立的prometheus服务器端。

具体搭建步骤如下：

* 编写prometheus.yaml

  此文件为prometheus标准配置文件，详细可参考[prometheus配置文件文档](https://www.prometheus.wang/configuration/)。
  这里，我们仅配置了metrics收集，并不使用alert等配置项。

  ```yaml
  # my global config
  global:
  scrape_interval: 5s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 5s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Alertmanager configuration
  alerting:
  alertmanagers:
    - static_configs:
        - targets:
        # - alertmanager:9093

  # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
  rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

  # A scrape configuration containing exactly one endpoint to scrape:
  # Here it's Prometheus itself.
  scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

      # metrics_path defaults to '/metrics'
      # scheme defaults to 'http'.

      static_configs:
      - targets: ["localhost:9090"]

  - job_name: "f5-kic-1-18"
      static_configs:
      - targets: ["10.xx.yy.zz:30080"]

  ```

  我们需要按照`job_name`的方式添加需要收集metrics的target。 这里的10.xx.yy.zz:30080即为CIS-C deployment暴露的prometheus metrics监听端口。

  CIS-C deployment暴露prometheus metrics监听端口的方式如下：

  ```yaml
  # expose the Prometheus port with NodePort
  apiVersion: v1
  kind: Service
  metadata:
    name: k8s-bigip-ctlr-c-svc
    namespace: kube-system
  spec:
    selector:
      app: k8s-bigip-ctlr-c-pod
    ports:
      - name: k8s-bigip-ctlr-c-metrics
        protocol: TCP
        port: 8080
        targetPort: 8080
        nodePort: 30080
  ```

* 编写docker-compose.yaml 

  这里，我们将本地的prometheus.yaml 文件挂载入container中；web端口9090。

  ```yaml
  version: "3"
  services:
  prometheus_on_k8s_master:
    image: prom/prometheus:latest
    volumes:
      - /root/prometheus/prometheus.yaml:/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090
  ```


## 数据数据采集

目前CIS-C通过Prometheus收集了业务下发过程中各个关键节点的时耗数据。

> CIS-C 暴露的metrics收集API为`/stats`，而不是默认的`/metrics`.

各个metrics名称及含义如下：

* bigip_icontrol_timecost_count： CIS-C下发过程中，发送iControlRest API到BIG-IP数量累计统计

  其中各个label的含义如下：

  | label      | 含义 |
  | ----------- | ----------- |
  |"method" |  REST请求方法：GET POST DELETE PATCH|
  |"url" | BIG-IP url名称，以“/mgmt/tm/..”为前缀 |

  例如：
  
  ```
  bigip_icontrol_timecost_count{method="GET",url="/mgmt/tm/ltm/snatpool"} 273
  ```
  
  表示CIS-C发送`/mgmt/tm/ltm/snatpool`请求的数量为`273`，这个数字也包括了对某snatpool对象的操作API，例如`DELETE /mgmt/tm/ltm/snatpool/~partitionA~subfolderB~snatpoolC`。

* bigip_icontrol_timecost_total： CIS-C下发过程中，发送iControlRest API到BIG-IP耗时累计统计

  其中各个label的含义如下：

  | label      | 含义 |
  | ----------- | ----------- |
  | "method" | REST请求方法：GET POST DELETE PATCH
  |"url" | BIG-IP url名称，以“/mgmt/tm/..”为前缀

  举例同上，单位为毫秒(ms)。

* function_duration_timecost：函数级别耗时统计。

  其中 label `name`为函数名称。

  此metrics主要用作追踪性能测试过程中的函数时耗情况，让开发者尽早发现当资源数量增加时的性能瓶颈点。


## 性能测试数据及方法

将CIS-C启动后，我们就可以做性能测试了，CIS-C启动方法、参数请参考 [下载与安装](./installation.md)。

性能测试过程中，我们使用ansible 脚本**逐个**创建多个资源。

以下代码会逐个创建100个资源。

```shell
for n in {0..100}; do 
    ansible-playbook \
        -e src=1.standalone.complex.resources.yaml.j2 \
        -e index=$n -e count=$(($n+1)) \
        -e action=apply \
        2.0.crud-in-batch.yaml
done
```

其中

* `2.0.crud-in-batch.yaml` 用于编排kubectl的资源yaml文件并调用kubectl创建资源。

  实现内容如下：

  ```yaml
  ---

  - hosts: localhost
    gather_facts: false
    remote_user: root
    vars_files:
        - 0.7.test-vars.json
    tasks:
        - name: generate resource yaml
        template:
            src: '{{ src }}'
            dest: ./tmp-resources.yaml

        - name: executing {{ action }}
        shell: |
            kubectl {{ action }} -f tmp-resources.yaml
    ```

* `1.standalone.complex.resources.yaml.j2` 为jinja2 模板文件：

  ```yaml
  {% for num in range(index|int, count|int, 1) %}

  ---

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: f5-vxlan-test-d{{ num }}
    namespace: default
  spec:
    replicas: 3
    selector:
        matchLabels:
        run: f5-vxlan-test-d{{ num }}
    template:
        metadata:
        labels:
            run: f5-vxlan-test-d{{ num }}
        spec:
        containers:
        - image: "nginx:latest"
            imagePullPolicy: IfNotPresent
            name: f5-vxlan-test-d{{ num }}
            ports:
            - containerPort: 80
            protocol: TCP


  ---

  apiVersion: v1
  kind: Service
  metadata:
    name: f5-vxlan-test-s{{ num }}
    namespace: default
    labels:
        cis.f5.com/as3-tenant: Tenant-{{ num }}
        cis.f5.com/as3-app: my-app
        cis.f5.com/as3-pool: mypool1
  spec:
    ports:
    - port: 80
        protocol: TCP
        targetPort: 80
    selector:
        run: f5-vxlan-test-d{{ num }}
    sessionAffinity: None
    type: ClusterIP


  ---

  apiVersion: v1
  kind: ConfigMap
  metadata:
    labels:
      f5type: virtual-server
      as3: "true"
    name: f5-vxlan-test-c{{ num }}
    namespace: default
  data:
    template: |
        {
            "class": "AS3",
            "declaration": {
                "class": "ADC",
                "schemaVersion": "3.18.0",
                "id": "urn:uuid:33045210-3ab8-4636-9b2a-c98d22ab915d",
                "label": "http",
                "remark": "application {{ num }} Template",
                "Tenant-{{ num }}": {
                    "class": "Tenant",
                    "my-app": {
                        "class": "Application",
                        "template": "generic",
                        "foo_ns_vs": {
                            "class": "Service_HTTP",
                            "remark": "app in Tenant-{{ num }}",
                            "virtualAddresses": [
                                "10.250.{{ 100 + (num/254)|int }}.{{ 1 + num%254 }}"
                            ],
                            "profileHTTP": {
                                "use": "http_X-Forwarded-For"
                            },
                            "persistenceMethods": [
                                {
                                    "use": "cookie_encryption"
                                }
                            ],
                            "iRules": [
                                "myirule"
                            ],
                            "pool": "mypool1"
                        },
                        "myirule": {
                            "class": "iRule",
                            "remark": "switch between pools",
                            "iRule": "when HTTP_REQUEST {\n if { [HTTP::uri] starts_with \"/coffee\" } {\n pool /Tenant-{{ num }}/my-app/mypool1 \n } else { \n pool /Tenant-{{ num }}/my-app/mypool2 \n } \n}"
                        },
                        "cookie_encryption": {
                            "class": "Persist",
                            "persistenceMethod": "cookie",
                            "encrypt": true,
                            "cookieMethod": "insert",
                            "passphrase": {
                                "ciphertext": "a3RjeGZ5Z2Q=",
                                "protected": "eyJhbGciOiJkaXIiLCJlbmMiOiJub25lIn0="
                            }
                        },
                        "http_X-Forwarded-For": {
                            "class": "HTTP_Profile",
                            "xForwardedFor": true
                        },
                        "mymonitor": {
                            "class": "Monitor",
                            "monitorType": "tcp",
                            "interval": 5,
                            "timeout": 16
                        },
                        "mypool1": {
                            "class": "Pool",
                            "monitors": [
                                "http"
                            ],
                            "members": [
                                {
                                    "servicePort": 80,
                                    "serverAddresses": []
                                }
                            ]
                        },
                        "mypool2": {
                            "class": "Pool",
                            "monitors": [
                                {
                                    "use": "mymonitor"
                                }
                            ],
                            "loadBalancingMode": "least-connections-member",
                            "minimumMembersActive": 0,
                            "members": [
                                {
                                    "servicePort": 80,
                                    "serverAddresses": []
                                }
                            ]
                        }
                    }
                }
            }
        }
  {% endfor %}
  ```

## 测试数据与结果样例

测试脚本执行结束后，就可以在prometheus的UI上查看测试结果了，具体的聚合PromSQL为：

* function_duration_timecost_count & function_duration_timecost_total

  ```shell
  # 查看函数增量调用次数
  deriv(function_duration_timecost_count[1m])
  # 查看函数增量耗时
  deriv(function_duration_timecost_total[1m])
  ```


* bigip_icontrol_timecost_count & bigip_icontrol_timecost_total

  ```shell
  # 查看icontrol调用次数
  bigip_icontrol_timecost_count
  # 查看icontrol调用耗时累计耗时
  bigip_icontrol_timecost_total
  ```

* deploy_request_timecost_count & deploy_request_timecost_count
  
  ```shell
  # 查看各种request调用增量次数
  deriv(deploy_request_timecost_count[1m])
  # 查看各种request调用增量耗时
  deriv(deploy_request_timecost_total[1m])
  ```