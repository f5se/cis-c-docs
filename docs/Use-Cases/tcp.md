## 1.TCP业务发布范例


### 场景描述

TCP业务发布

示例中创建了定制化的TCP profile：customTCPProfile，
且创建了一个snatpool：CreateSnatPool。
请视需要需要相关参数。

### 参考yaml

~~~
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-as3-tcp-std-template-configmap
  namespace: default
  labels:
    f5type: virtual-server
    as3: "true"
data:
  template: |
    {
        "class": "AS3",
        "action": "deploy",
        "persist": true,
        "declaration": {
            "class": "ADC",
            "schemaVersion": "3.28.0",
            "label": "snat_pool_existing",
            "remark": "reference_snat_pool",
            "Sample_TCP_Pool": {
                "class": "Tenant",
                "TCP_app": {
                    "class": "Application",
                    "test_service": {
                        "class": "Service_TCP",
                        "virtualPort": 8181,
                        "virtualAddresses": [
                            "10.42.20.120"
                        ],
                        "profileTCP": {
                            "use": "customTCPProfile"
                        },
                        "pool": "tcp_pool",
                        "snat": {
                            "use": "CreateSnatPool"
                        }
                    },
                    "customTCPProfile": {
                        "class": "TCP_Profile",
                        "idleTimeout": 600
                    },
                    "tcp_pool": {
                        "class": "Pool",
                        "monitors": [
                            "tcp"
                        ],
                        "members": [
                            {
                                "servicePort": 80,
                                "serverAddresses": []
                            }
                        ]
                    },
                    "CreateSnatPool": {
                        "class": "SNAT_Pool",
                        "snatAddresses": [
                            "10.42.20.192",
                            "10.42.20.193"
                        ]
                    }
                }
            }
        }
    }
~~~