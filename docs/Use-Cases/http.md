# 标准HTTP业务发布范例


## 场景描述

标准HTTP业务发布

HTTP业务默认参数说明：
Snat为auto，意思是automap
persistenceMethods为cookie，意思默认会话保持是cookie
VS类型为standard
默认会使用http profile

## 参考yaml

~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-11
  namespace: f5-test005
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
        "remark": "HTTP application",
        "test005": {
          "class": "Tenant",
          "test005": {
            "class": "Application",
            "template": "generic",
            "pool3": {
              "class": "Pool",
              "monitors": [
                "tcp"
              ],
              "members": [
              {
                "servicePort": 8080,
                "serverAddresses": [ ]
              }
              ]
            },                           
            "app_svc_vs1": {
              "class": "Service_HTTP",
              "virtualAddresses": [
                "172.16.142.181"
              ],
              "virtualPort": 82,
              "pool": "pool3"              
            }
          }
        }
      }
    }
~~~

## 部署结果

~~~
list ltm virtual test005/app_svc_vs1 

ltm virtual test005/app_svc_vs1 {
    creation-time 2022-08-02:16:28:32
    description test005
    destination 172.16.142.181:xfer
    ip-protocol tcp
    last-modified-time 2022-08-02:16:28:32
    mask 255.255.255.255
    partition test005
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool test005/pool3
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 933
}
~~~



# 配置virtual address属性范例
## 场景描述
配置virtual address对象, arp enable no， virtual server 引用该对象

## 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-2
  namespace: f5-test005
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
        "remark": "HTTP application",
        "test005": {
          "class": "Tenant",
          "test005": {
            "class": "Application",
            "template": "generic",
            "pool3": {
              "class": "Pool",
              "monitors": [
                "tcp"
              ],
              "members": [
              {
                "servicePort": 8080,
                "serverAddresses": [ ]
              }
              ]
            },                     
            "app_svc_vip": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.178",
              "arpEnabled": false
            },        
            "app_svc_vs2": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip" }],
              "virtualPort": 82,
              "pool": "pool3"              
            }
          }
        }
      }
    }
~~~    

## 部署结果

~~~

list ltm virtual test005/app_svc_vs2

ltm virtual test005/app_svc_vs2 {
    creation-time 2022-08-02:16:43:17
    description test005
    destination 172.16.142.178:xfer
    ip-protocol tcp
    last-modified-time 2022-08-02:16:43:17
    mask 255.255.255.255
    partition test005
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool test005/pool3
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 934
}

list ltm virtual-address 172.16.142.178 

ltm virtual-address 172.16.142.178 {
    address 172.16.142.178
    arp disabled
    inherited-traffic-group true
    mask 255.255.255.255
    partition test005
    traffic-group /Common/traffic-group-1
}
~~~

# 配置TCP profile范例
## 场景描述
配置TCP profile，自定义超时时间

## 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-3
  namespace: f5-test005
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
        "remark": "HTTP application",
        "test005": {
          "class": "Tenant",
          "test005": {
            "class": "Application",
            "template": "generic",
            "customTCPProfile": {
              "class": "TCP_Profile",
              "idleTimeout": 600
            },
            "pool3": {
              "class": "Pool",
              "monitors": [
                "tcp"
              ],
              "members": [
              {
                "servicePort": 8080,
                "serverAddresses": [ ]
              }
              ]
            },
            "app_svc_vip": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.178",
              "arpEnabled": false
            },
            "app_svc_vs2": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip" }],
              "virtualPort": 82,
              "profileTCP": {
                "use": "customTCPProfile"
              },
              "pool": "pool3"
            }
          }
        }
      }
    }
~~~    

## 部署结果

~~~

list ltm virtual test005/app_svc_vs2 

ltm virtual test005/app_svc_vs2 {
    creation-time 2022-08-02:17:04:31
    description test005
    destination 172.16.142.178:xfer
    ip-protocol tcp
    last-modified-time 2022-08-02:17:04:31
    mask 255.255.255.255
    partition test005
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool test005/pool3
    profiles {
        /Common/http { }
        test005/customTCPProfile { }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 935
}

list ltm profile tcp test005/customTCPProfile | grep idle-timeout

    idle-timeout 600

~~~

# 配置HTTP profile范例
## 场景描述
配置HTTP profile，开启XFF

## 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-4
  namespace: f5-test006
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
        "remark": "HTTP application",
        "test006": {
          "class": "Tenant",
          "test006": {
            "class": "Application",
            "template": "generic",
            "customHTTPProfile": {
              "class": "HTTP_Profile",
              "xForwardedFor": true
              },
            "pool3": {
              "class": "Pool",
              "monitors": [
                "tcp"
              ],
              "members": [
              {
                "servicePort": 8080,
                "serverAddresses": [ ]
              }
              ]
            },
            "app_svc_vip4": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.188",
              "arpEnabled": false
            },
            "app_svc_vs2": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip4" }],
              "virtualPort": 82,
              "profileHTTP": {
                "use": "customHTTPProfile"
              },
              "pool": "pool3"
            }
          }
        }
      }
    }
~~~    

## 部署结果

~~~

list ltm virtual test006/app_svc_vs2 

ltm virtual test006/app_svc_vs2 {
    creation-time 2022-08-02:17:57:57
    description test006
    destination 172.16.142.188:xfer
    ip-protocol tcp
    last-modified-time 2022-08-02:17:57:57
    mask 255.255.255.255
    partition test006
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool test006/pool3
    profiles {
        /Common/f5-tcp-progressive { }
        test006/customHTTPProfile { }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 946
}

list ltm profile http test006/customHTTPProfile | grep "insert-xforwarded-for"

    insert-xforwarded-for enabled

~~~




```
