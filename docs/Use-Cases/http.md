## 1.标准HTTP业务发布范例


### 场景描述

标准HTTP业务发布

HTTP业务默认参数说明：
Snat为auto，意思是automap
persistenceMethods为cookie，意思默认会话保持是cookie
VS类型为standard
默认会使用http profile

### 参考yaml

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

### 部署结果

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



## 2.配置virtual address属性范例
### 场景描述
配置virtual address对象, arp enable no， virtual server 引用该对象

### 参考yaml
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

###  部署结果

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

## 3.配置TCP profile范例
### 场景描述
配置TCP profile，自定义超时时间

### 参考yaml
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

### 部署结果

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

## 4.配置HTTP profile范例
### 场景描述
配置HTTP profile，开启XFF

### 参考yaml
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

### 部署结果

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



## 5.配置cookie会话保持，cookie加密范例
### 场景描述
配置cookie会话保持，对F5 insert的cookie进行加密

### 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-6
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
            "app_svc_vip6": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.186",
              "arpEnabled": false
            },        
            "app_svc_vs6": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip6" }],
              "persistenceMethods": [{ "use": "cookie_encryption" }],          
              "virtualPort": 82,
              "pool": "pool3"              
            }
          }
        }
      }
    }
~~~    

### 部署结果

~~~

list ltm virtual test006/app_svc_vs6 

ltm virtual test006/app_svc_vs6 {
    creation-time 2022-08-03:09:31:30
    description test006
    destination 172.16.142.186:xfer
    ip-protocol tcp
    last-modified-time 2022-08-03:09:31:30
    mask 255.255.255.255
    partition test006
    persist {
        test006/cookie_encryption {
            default yes
        }
    }
    pool test006/pool3
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
    vs-index 939
}

list ltm persistence cookie test006/cookie_encryption 

ltm persistence cookie test006/cookie_encryption {
    always-send disabled
    app-service none
    cookie-encryption required
    cookie-encryption-passphrase $M$t7$uQ0K+M40jOAbMER6IY+BSA==
    cookie-name none
    expiration 0
    httponly enabled
    match-across-pools disabled
    match-across-services disabled
    match-across-virtuals disabled
    method insert
    mirror disabled
    override-connection-limit disabled
    secure enabled
    timeout indefinite
}

~~~



## 6.配置snat pool范例
## 场景描述
配置snat pool范例

### 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-6
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
            "snatpool-3": {
              "class": "SNAT_Pool",
              "snatAddresses": [
              "172.16.142.21",
              "172.16.142.22"
              ]
            },                               
            "app_svc_vip6": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.186",
              "arpEnabled": false
            },        
            "app_svc_vs6": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip6" }],
              "snat": {
                "use": "snatpool-3"
              },          
              "virtualPort": 82,
              "pool": "pool3"              
            }
          }
        }
      }
    }
~~~    

### 部署结果

~~~

list ltm virtual test006/app_svc_vs6

ltm virtual test006/app_svc_vs6 {
    creation-time 2022-08-03:09:36:04
    description test006
    destination 172.16.142.186:xfer
    ip-protocol tcp
    last-modified-time 2022-08-03:09:36:04
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
        /Common/http { }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        pool test006/snatpool-3
        type snat
    }
    translate-address enabled
    translate-port enabled
    vs-index 940
}

list ltm snatpool test006/snatpool-3 

ltm snatpool test006/snatpool-3 {
    members {
        172.16.142.21
        172.16.142.22
    }
    partition test006
}

~~~

## 7.配置one connect范例
### 场景描述
配置one connect范例

### 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-6
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
            "customOne": {
              "class": "Multiplex_Profile",
              "sourceMask": "ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff"
            },                              
            "app_svc_vip6": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.186",
              "arpEnabled": false
            },        
            "app_svc_vs6": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip6" }],
              "profileMultiplex": {
                "use": "customOne"
              },         
              "virtualPort": 82,
              "pool": "pool3"              
            }
          }
        }
      }
    }
~~~    

### 部署结果

~~~

list ltm virtual test006/app_svc_vs6
ltm virtual test006/app_svc_vs6 {
    creation-time 2022-08-03:09:40:20
    description test006
    destination 172.16.142.186:xfer
    ip-protocol tcp
    last-modified-time 2022-08-03:09:40:20
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
        /Common/http { }
        test006/customOne { }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 941
}

list ltm profile one-connect test006/customOne 

ltm profile one-connect test006/customOne {
    app-service none
    idle-timeout-override disabled
    limit-type none
    max-age 86400
    max-reuse 1000
    max-size 10000
    share-pools disabled
    source-mask ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff
}

~~~

## 8.配置VS 链接限制， immediate action on svc down范例
### 场景描述
配置VS 链接限制， immediate action on svc down

### 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-6
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
            "app_svc_vip6": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.186",
              "arpEnabled": false
            },        
            "app_svc_vs6": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip6" }],
              "virtualPort": 82,
              "serviceDownImmediateAction": "reset",
              "maxConnections": 123,              
              "pool": "pool3"              
            }
          }
        }
      }
    }
~~~    

### 部署结果

~~~

list ltm virtual test006/app_svc_vs6

ltm virtual test006/app_svc_vs6 {
    connection-limit 123
    creation-time 2022-08-03:09:51:12
    description test006
    destination 172.16.142.186:xfer
    ip-protocol tcp
    last-modified-time 2022-08-03:09:51:12
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
        /Common/http { }
    }
    serverssl-use-sni disabled
    service-down-immediate-action reset
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 943
}
~~~



## 9.配置VS关联多个iRules范例
### 场景描述
配置VS关联多个iRules

### 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-6
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
            "iRulesTest1": {
              "class": "iRule",
              "iRule": "when CLIENT_ACCEPTED {\n \n }"
            },
            "iRulesTest2": {
              "class": "iRule",
              "iRule": "when CLIENT_ACCEPTED {\n \n }"
            },                                         
            "app_svc_vip6": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.186",
              "arpEnabled": false
            },        
            "app_svc_vs6": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip6" }],
              "iRules": [{ "use": "iRulesTest1" },{ "use": "iRulesTest2" }],                       
              "virtualPort": 82,
              "pool": "pool3"              
            }
          }
        }
      }
    }
~~~    

### 部署结果

~~~

list ltm virtual test006/app_svc_vs6

ltm virtual test006/app_svc_vs6 {
    creation-time 2022-08-03:09:46:05
    description test006
    destination 172.16.142.186:xfer
    ip-protocol tcp
    last-modified-time 2022-08-03:09:46:05
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
        /Common/http { }
    }
    rules {
        test006/iRulesTest1
        test006/iRulesTest2
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 942
}
list ltm rule test006/iRulesTest1 

ltm rule test006/iRulesTest1 {
    partition test006
when CLIENT_ACCEPTED {
 
 }
}

list ltm rule test006/iRulesTest2

ltm rule test006/iRulesTest2 {
    partition test006
when CLIENT_ACCEPTED {
 
 }
}
~~~

## 10.配置多个monitor范例
### 场景描述
pool中配置多个monitor

### 参考yaml
~~~
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-as3-6
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
            "monitorTcp1": {
              "class": "Monitor",
              "monitorType": "tcp",
              "timeout": 10,
              "interval": 2,
              "send": "",
              "receive": ""
            },
            "monitorTcp2": {
              "class": "Monitor",
              "monitorType": "tcp",
              "timeout": 7,
              "interval": 2,
              "send": "",
              "receive": ""
            },            
            "pool3": {
              "class": "Pool",
              "monitors": [
                {
                  "use": "monitorTcp1"
                },
                {
                  "use": "monitorTcp2"
                }                
              ],
              "minimumMonitors": "all",
              "members": [
              {
                "servicePort": 8080,
                "serverAddresses": [ ]
              }
              ]
            },                                      
            "app_svc_vip6": {
              "class": "Service_Address",
              "virtualAddress": "172.16.142.186",
              "arpEnabled": false
            },        
            "app_svc_vs6": {
              "class": "Service_HTTP",
              "virtualAddresses": [{ "use": "app_svc_vip6" }],
              "virtualPort": 82,
              "serviceDownImmediateAction": "reset",
              "maxConnections": 123,              
              "pool": "pool3"              
            }
          }
        }
      }
    }

~~~    

### 部署结果

~~~
list ltm pool test006/pool3 

ltm pool test006/pool3 {
    min-active-members 1
    monitor test006/monitorTcp1 and test006/monitorTcp2
    partition test006
}
~~~

## 11.常用irules范例


### 根据URI进行转发

```
"irule_k8s-cib001-svc-1": {
  "class": "iRule",
  "remark": "switch between pools",
  "iRule": "when HTTP_REQUEST {\n if { [HTTP::uri] starts_with \"/coffee\" } {\n pool k8s-cib001-svc-1-pool-8080 \n } else { \n pool k8s-cib001-app-svc-4-pool-8080 \n }\n  \n}" 
},
```
其中`if { [HTTP::uri] starts_with \"/coffee\" }`表示新的版本中以/coffee目录开头的都会分往新版本的服务器上，`{\n pool k8s-cib001-svc-1-pool-8080 \n }`其中的pool是新版本服务器的地址池
如果不是以/coffee开头的都会分往旧版本的服务器上


### 根据客户端ip进行转发
```
"irule_k8s-cib001-svc-1": {
  "class": "iRule",
  "remark": "switch between pools",
  "iRule": "when CLIENT_ACCEPTED {\n  if {[IP::addr [IP::client_addr] equals 192.168.5.30] or [IP::addr [IP::client_addr] equals 192.168.5.0/24]} {\n    pool k8s-cib001-svc-1-pool-8080 \n  } else {\n    pool k8s-cib001-app-svc-4-pool-8080 \n  }\n}"
},
```

其中当终端地址为192.168.5.30或者属于192.168.5.0/24的网段，该请求流量分给`pool k8s-cib001-svc-1-pool-8080`
其它的情况流量均分往`pool k8s-cib001-app-svc-1-pool-8080`
注意一下如上irule的写法，`\n`表示换行


### 根据cookie进行转发
```
"irule_k8s-cib001-svc-1": {
  "class": "iRule",
  "remark": "switch between pools",
  "iRule": "when HTTP_REQUEST {\n  if {[HTTP::cookie exists \"Canary\"] and [HTTP::cookie value \"Canary\"] equals \"true\"} {\n    pool k8s-cib001-svc-1-pool-8080 \n  } else {\n    pool k8s-cib001-app-svc-4-pool-8080 \n  }\n}"
},
```
其中如果可以存在cookie为Canary，而且对应该cookie值为true，流量分给`pool k8s-cib001-svc-1-pool-8080` 
其它的所有流量分往`pool k8s-cib001-app-svc-1-pool-8080`
注意一下如上irule的写法，`\n`表示换行



### 根据http header进行转发
```
"irule_k8s-cib001-svc-1": {
  "class": "iRule",
  "remark": "switch between pools",
  "iRule":  "when HTTP_REQUEST {\n  if {[HTTP::header exists \"Canary\"] and [HTTP::header values \"Canary\"] equals \"true\"} {\n    pool k8s-cib001-svc-1-pool-8080 \n  } else {\n    pool k8s-cib001-app-svc-4-pool-8080 \n  }\n}"
},
```
其中http header中如果存在\"Canary\"，且对应的value值为true，流量分给`pool k8s-cib001-svc-1-pool-8080`  
其它情况流量分给`k8s-cib001-app-svc-1-pool-8080`
注意一下如上irule的写法，`\n`表示换行


### 根据比例进行灰度
```
"irule_k8s-cib001-svc-1": {
  "class": "iRule",
  "remark": "switch between pools",
  "iRule": "when CLIENT_ACCEPTED {\n  if {[expr {[expr {0xffffffff & [crc32 [IP::client_addr]]}] % 100}] < 25} {\n    pool k8s-cib001-svc-1-pool-8080 \n  } else {\n    pool k8s-cib001-app-svc-4-pool-8080 \n  }\n}"
},
```
其中< 25 是 25%的流量分给`pool k8s-cib001-svc-1-pool-8080` 
75%的流量分往`pool k8s-cib001-app-svc-1-pool-8080`
注意一下如上irule的写法，`\n`表示换行