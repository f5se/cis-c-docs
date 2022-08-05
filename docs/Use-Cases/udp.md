## 1.UDP业务发布范例


### 场景描述

UDP业务发布

示例中创建了定制化的UDP profile：customUDPProfile，
且创建了一个snatpool：CreateSnatPool。
请视需要需要相关参数。

### 参考yaml

~~~
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-as3-udp-std-template-configmap
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
            "label": "udp-std-vs",
            "remark": "reference_snat_pool",
            "Sample_SNAT_Pool": {
                "class": "Tenant",
                "SNAT_app": {
                    "class": "Application",
                    "test_service": {
                        "class": "Service_UDP",
                        "virtualPort": 53,
                        "virtualAddresses": [
                            "10.42.20.119"
                        ],
                        "profileUDP": {
                            "use": "customUDPProfile"
                        },
                        "pool": "udp_pool",
                        "snat": {
                            "use": "CreateSnatPool"
                        }
                    },
                    "customUDPProfile": {
                        "class": "UDP_Profile",
                        "idleTimeout": 10
                    },
                    "udp_pool": {
                        "class": "Pool",
                        "monitors": [
                            "icmp"
                        ],
                        "members": [
                            {
                                "servicePort": 80,
                                "serverAddresses": ["1.1.1.1"]
                            }
                        ]
                    },
                    "CreateSnatPool": {
                        "class": "SNAT_Pool",
                        "snatAddresses": [
                            "10.42.20.190",
                            "10.42.20.191"
                        ]
                    }
                }
            }
        }
    }
~~~