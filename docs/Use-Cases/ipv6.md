## 1.IPv6业务发布范例


### 场景描述

IPv6业务发布

本示例中指定了virtualAddresses为一个IPv6地址（Configmap中的其余部分可参考对应类型的configmap示例）.
最终 CIS-C 会将该IPv6地址作为virtual server 的地址下发到BIGIP上。
若configmap相关的pool member的地址也是IPv6类型，则CIS-C也会将其IPv6地址作为member的地址进行下发。
请视需要修改相关参数。

### 参考yaml

~~~
apiVersion: v1
kind: ConfigMap
metadata:
  name: f5-vxlan-test-c1000
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
            "id": "veconfig-nsgroup1",
            "namespace1000": {
                "class": "Tenant",
                "service_name_app1000": {
                    "class": "Application",
                    "template": "generic",
                    "service_name_vs8": {
                        "class": "Service_TCP",
                        "virtualAddresses": [
                            "2001:db8:42:1::33"
                        ],
                        "virtualPort": 8008,
                        "profileFTP":  {
                            "use": "service_name_ftpprofile8"
                        },
                        "snat": "self",
                        "pool": "service_name_pool1000"
                    },
                    "service_name_pool1000": {
                        "class": "Pool",
                        "monitors": [
                            "tcp"
                        ],
                        "loadBalancingMode": "least-connections-member",
                        "members": [
                            {
                                "servicePort": 80,
                                "serverAddresses": []
                            }
                        ]
                    },
                    "service_name_ftpprofile8": {
                        "class": "FTP_Profile",
                        "remark": "description",
                        "port": 300,
                        "ftpsMode": "allow",
                        "enforceTlsSessionReuseEnabled": true,
                        "activeModeEnabled": false,
                        "securityEnabled": true,
                        "translateExtendedEnabled": false,
                        "inheritParentProfileEnabled": true
                    }
                }
            }
        }
      }
~~~

### 部署结果示例

~~~
list ltm virtual

ltm virtual service_name_vs8 {
    creation-time 2022-08-10:02:10:29
    description service_name_app1000
    destination /namespace1000/2001:db8:42:1::33.8008
    ip-protocol tcp
    last-modified-time 2022-08-10:02:10:29
    partition namespace1000
    persist {
        /Common/source_addr {
            default yes
        }
    }
    pool service_name_pool1000
    profiles {
        /Common/f5-tcp-progressive { }
        service_name_ftpprofile8 { }
    }
    source-address-translation {
        pool service_name_vs8_self
        type snat
    }
    translate-address enabled
    translate-port enabled
    vs-index 4
}

~~~