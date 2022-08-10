## 1.FTP业务发布范例


### 场景描述

FTP业务发布

示例中创建了定制化的FTP profile：service_name_ftpprofile .
请视需要修改相关参数。

### 参考yaml

~~~
apiVersion: v1
kind: ConfigMap
metadata:
  name: f5-ftp-test
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
            "ftptest": {
                "class": "Tenant",
                "app-1": {
                    "class": "Application",
                    "template": "generic",
                    "app_service_ftp": {
                        "class": "Service_TCP",
                        "virtualAddresses": [
                            "10.42.20.118"
                        ],
                        "virtualPort": 21,
                        "profileFTP":  {
                            "use": "service_name_ftpprofile"
                        },
                        "snat": "self",
                        "pool": "service_name_ftp_pool"
                    },
                    "service_name_ftp_pool": {
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
                    "service_name_ftpprofile": {
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