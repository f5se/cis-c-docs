## 1.标准HTTPS业务发布范例

### 场景描述

最常用的HTTPS业务发布，前端使用HTTPS，导入并配置服务器证书和密钥，后端使用HTTP，实现SSL Offloading。

### 参考YAML

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-6
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
        "schemaVersion": "3.27.0",
        "remark": "Normal https",
        "cis-c-6": {
          "class": "Tenant",
          "cafe": {
            "class": "Application",
            "https_vs": {
              "class": "Service_HTTPS",
              "virtualAddresses": [
                "192.0.2.11"
              ],
              "virtualPort": 443,
              "pool": "coffee_pool",
              "serverTLS": "coffeetls"
            },
            "coffee_pool": {
              "class": "Pool",
              "monitors": [
                "http"
              ],
              "members": [{
                "servicePort": 80,
                "serverAddresses": []
              }]
            },
            "coffeetls": {
              "class": "TLS_Server",
              "certificates": [{
                "certificate": "coffeecert"
              }]
            },
            "coffeecert": {
              "class": "Certificate",
              "remark": "coffee.example.com",
              "certificate": "-----BEGIN CERTIFICATE-----\nMIIDPDCCAiSgAwIBAgIEF6x2/TANBgkqhkiG9w0BAQsFADBgMQswCQYDVQQGEwJD\nTjELMAkGA1UECBMCWkoxCzAJBgNVBAcTAkhaMQswCQYDVQQKEwJGNTENMAsGA1UE\nCxMEVGVzdDEbMBkGA1UEAxMSY29mZmVlLmV4YW1wbGUuY29tMB4XDTIyMDgwMzA3\nMjM0MVoXDTMyMDczMTA3MjM0MVowYDELMAkGA1UEBhMCQ04xCzAJBgNVBAgTAlpK\nMQswCQYDVQQHEwJIWjELMAkGA1UEChMCRjUxDTALBgNVBAsTBFRlc3QxGzAZBgNV\nBAMTEmNvZmZlZS5leGFtcGxlLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC\nAQoCggEBAOg2bcgZP1hCWIGnWikq8hivZq130nfXsyDijf0VpkvfHDmVdfEIyz9k\nhRVVgCTtw5jllGsGZ4O+0jEx+bGGwAUgztH/318IW+EUFd928jaUwgYiWqSwbtgk\nHjEUH30U9bXz1nvFISOjU33imbJsDq4Rjvq3/YxelMeRFw0xgMAWiyEFnbVU41cQ\nFP6+PqZbJ1/wZ4nhTWnJGmYvEmtQ2Fh27JGQjkqrKp22PV8c8tds8+CyCbi/6zOR\nJExj2zQ/zuOIVgm26z75OSsuRf+W7dFA0Li6zUdk7y1iw3Y/yI4I+htfORTum8SM\nzG99ssbuE2lNQJ2Zh4tVz9bHwOU2+p8CAwEAATANBgkqhkiG9w0BAQsFAAOCAQEA\nQrzflgFiNs1pA4ou/1+q2o59/cw6ga5MXWbjVDCh34w9okpzNnEmPlBlvLiLykSV\n5H7u6pnNP1EUPdDe+Cleg0E2Om0pIwuvmBc8YT8AADfE+znGb/OUEQOZ4pGSbxeQ\nZX5/H5Ie4UszfcEPfNnBerMRX6OBy39RjIQBTvioSMCu4agfzY8eubQfDwBpRb/o\n+hOh2IB/fnr7zxbs7qSBiTktXsspfa3nezrdNQ+iXdTRPrBit+2j67CrvKpBoRKe\nauY+7woPVDVJYjeAmy5Ly8zZUoGDlpRwKEtTu48108Cg3bISLhqycX18ZpZ2BA1N\nu+QU1Yn3RaGFSU/8DCJxtQ==\n-----END CERTIFICATE-----",
              "privateKey": "-----BEGIN PRIVATE KEY-----\nMIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQDoNm3IGT9YQliB\np1opKvIYr2atd9J317Mg4o39FaZL3xw5lXXxCMs/ZIUVVYAk7cOY5ZRrBmeDvtIx\nMfmxhsAFIM7R/99fCFvhFBXfdvI2lMIGIlqksG7YJB4xFB99FPW189Z7xSEjo1N9\n4pmybA6uEY76t/2MXpTHkRcNMYDAFoshBZ21VONXEBT+vj6mWydf8GeJ4U1pyRpm\nLxJrUNhYduyRkI5Kqyqdtj1fHPLXbPPgsgm4v+szkSRMY9s0P87jiFYJtus++Tkr\nLkX/lu3RQNC4us1HZO8tYsN2P8iOCPobXzkU7pvEjMxvfbLG7hNpTUCdmYeLVc/W\nx8DlNvqfAgMBAAECggEASSUH7J1DTkZLwb9Yz5nm+26YrbCOG9DWbFfguOUuZjzH\nk73oEj4eY4ACyacOf9NjJtC+MP4p8h5T8EoZKFnVN2hPrWdnUXR9GIduol7Byf6O\npUcB/VlT+QJbfkMj7g8BnMhLed4s46BpRsBvgHu4Hg2K15/IHoSWYcxqke3Ta+20\nQo/cJxfgr5lHtae6XXBnp/Mu+vMuQZxFAz5TR6bWPjwRqyok8Mk1JK9we0nEJm8U\nml66N9i0UerhvIyFXUMww+846J/HnyrRvz0j5EnYmcquDw3DMMXzpssYCzamueny\nMgiD6VujfT6pQ/ClQQHZf3+naDPA6dz4zeVOwxvquQKBgQD8FNvObw4TI2ZRNlIW\nv6By4JWAIut25NI6r6+UXMYHbKxP7bI97P1SQMV+kGdVxiKJ366BM18WlzOIurJy\nFKtE6QaggEZYN/duotA+jt3MygyXg2gu2wEpBzIwgxxRbR05pQLlsLtbSxKlpfZV\nk5trX0ZQ2eaUw0ITb43rsJ7VdwKBgQDr0oDeQv8FPCbsmgGeX0UWLP1aBeCJZNub\n15E5vXAy22fmsJTuAh7kvyyEHe4c/maug44VFEahn2H+6H2YGdzZTmGHzcxDpjSe\nIVQswxNCxpdgo/w5aR7M9Ewn5ollZotO0eIzj0MVaETW6UBqYX8oRgicpHC4KrKM\nGMabZJ1uGQKBgQCUZts4XpzUm4SCzw3ooouc1aZttyET74XsUr11BGD4wft3WqIS\nXtCLeeJKrkyHbIusy2h6W3nhXMZT+kVPb+ecO+tQ1fOTv+8EzQj3qzfcdh6PnCbb\nXscCFmBvuuAS97+6zfA0tKS4DCxAJMIugyV+QqqssntSnNjrhELyvBnl5QKBgQCk\nx+ioZiQQomGIfmyXH3cE8dbuaqDlIIabtNuTfx3BS7KkbcsDLJQtvq/6eXeC5vkV\nBHPpostf8CDnn8jy2U+KwMxBurn6o06tGBjbVkxFIsNwEeYSr7OH/0SftOVY53h8\nUQhAguCbOsqvaTlLnGjf2V/3JKhm597vKfjNaFbhMQKBgQCb7+WkDN9Pc3n1gbo3\nnZW7sTasIDj0MoZuMiM8J6R9EsD+hNJ/c5vpr4CeFaT4DTanhqLR6cCPR+Nqcl1T\neB4IKarz04svwizzFSG0XDCVmKO3nM4Zai0zkAIFSZxIXG9pxmucOmx8DPrvlX/c\nptO8AvBlmMvMiGlAMyDlKMUYSQ==\n-----END PRIVATE KEY-----"
            }
          }
        }
      }
    }
```

### 部署结果

```
ltm virtual cafe/https_vs {
    creation-time 2022-08-03:15:25:38
    description cafe
    destination 192.0.2.11:https
    ip-protocol tcp
    last-modified-time 2022-08-03:15:25:38
    mask 255.255.255.255
    partition cis-c-6
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool cafe/coffee_pool
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
        cafe/coffeetls {
            context clientside
        }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 41
}


ltm profile client-ssl cafe/coffeetls {
    alert-timeout indefinite
    allow-expired-crl disabled
    app-service none
    authenticate once
    c3d-drop-unknown-ocsp-status drop
    cache-timeout 3600
    cert cafe/coffeecert.crt
    cert-key-chain {
        coffeecert {
            cert cafe/coffeecert.crt
            key cafe/coffeecert.key
        }
    }
    cert-lookup-by-ipaddr-port disabled
    cipher-group none
    ciphers DEFAULT
    inherit-ca-certkeychain true
    inherit-certkeychain false
    key cafe/coffeecert.key
    ocsp-stapling disabled
    peer-cert-mode ignore
    renegotiation enabled
    retain-certificate true
    sni-require false
    ssl-c3d disabled
    ssl-forward-proxy disabled
    ssl-forward-proxy-bypass disabled
}


sys crypto cert cafe/coffeecert.crt {
    cert-validation-options none
    cert-validators {
         { }
    }
    certificate-key-size 2048
    city HZ
    common-name coffee.example.com
    country CN
    email-address
    expiration Jul 31 07:23:41 2032 GMT
    fingerprint SHA256/D6:7A:CB:DA:2D:48:A1:21:27:15:39:5A:24:58:56:00:4E:53:A2:55:F8:AF:F1:DB:DF:07:AB:85:68:76:AD:AF
    issuer CN=coffee.example.com,OU=Test,O=F5,L=HZ,ST=ZJ,C=CN
    issuer-certificate
    organization F5
    ou Test
    public-key-type RSA
    state ZJ
    subject-alternative-name
}


sys crypto key cafe/coffeecert.key {
    key-size 2048
    key-type rsa-private
    security-type normal
}


ltm virtual cafe/https_vs-Redirect- {
    creation-time 2022-08-03:15:25:38
    description cafe
    destination 192.0.2.11:http
    ip-protocol tcp
    last-modified-time 2022-08-03:15:25:38
    mask 255.255.255.255
    partition cis-c-6
    pool cafe/coffee_pool
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
    }
    rules {
        /Common/_sys_https_redirect
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 42
}
```

请注意最后一个HTTP的VS，**cafe/https_vs-Redirect-**，这是AS3自动生成的，用于HTTP的跳转，如果不需要自动生成这个VS，请在AS3中加入以下参数：

`"redirect80": false`

完整的YAML如下：

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-6
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
        "schemaVersion": "3.27.0",
        "remark": "Normal https",
        "cis-c-6": {
          "class": "Tenant",
          "cafe": {
            "class": "Application",
            "https_vs": {
              "class": "Service_HTTPS",
              "virtualAddresses": [
                "192.0.2.11"
              ],
              "virtualPort": 443,
              "pool": "coffee_pool",
              "serverTLS": "coffeetls",
              "redirect80": false
            },
            "coffee_pool": {
              "class": "Pool",
              "monitors": [
                "http"
              ],
              "members": [{
                "servicePort": 80,
                "serverAddresses": []
              }]
            },
            "coffeetls": {
              "class": "TLS_Server",
              "certificates": [{
                "certificate": "coffeecert"
              }]
            },
            "coffeecert": {
              "class": "Certificate",
              "remark": "coffee.example.com",
              "certificate": "-----BEGIN CERTIFICATE-----\nMIIDPDCCAiSgAwIBAgIEF6x2/TANBgkqhkiG9w0BAQsFADBgMQswCQYDVQQGEwJD\nTjELMAkGA1UECBMCWkoxCzAJBgNVBAcTAkhaMQswCQYDVQQKEwJGNTENMAsGA1UE\nCxMEVGVzdDEbMBkGA1UEAxMSY29mZmVlLmV4YW1wbGUuY29tMB4XDTIyMDgwMzA3\nMjM0MVoXDTMyMDczMTA3MjM0MVowYDELMAkGA1UEBhMCQ04xCzAJBgNVBAgTAlpK\nMQswCQYDVQQHEwJIWjELMAkGA1UEChMCRjUxDTALBgNVBAsTBFRlc3QxGzAZBgNV\nBAMTEmNvZmZlZS5leGFtcGxlLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC\nAQoCggEBAOg2bcgZP1hCWIGnWikq8hivZq130nfXsyDijf0VpkvfHDmVdfEIyz9k\nhRVVgCTtw5jllGsGZ4O+0jEx+bGGwAUgztH/318IW+EUFd928jaUwgYiWqSwbtgk\nHjEUH30U9bXz1nvFISOjU33imbJsDq4Rjvq3/YxelMeRFw0xgMAWiyEFnbVU41cQ\nFP6+PqZbJ1/wZ4nhTWnJGmYvEmtQ2Fh27JGQjkqrKp22PV8c8tds8+CyCbi/6zOR\nJExj2zQ/zuOIVgm26z75OSsuRf+W7dFA0Li6zUdk7y1iw3Y/yI4I+htfORTum8SM\nzG99ssbuE2lNQJ2Zh4tVz9bHwOU2+p8CAwEAATANBgkqhkiG9w0BAQsFAAOCAQEA\nQrzflgFiNs1pA4ou/1+q2o59/cw6ga5MXWbjVDCh34w9okpzNnEmPlBlvLiLykSV\n5H7u6pnNP1EUPdDe+Cleg0E2Om0pIwuvmBc8YT8AADfE+znGb/OUEQOZ4pGSbxeQ\nZX5/H5Ie4UszfcEPfNnBerMRX6OBy39RjIQBTvioSMCu4agfzY8eubQfDwBpRb/o\n+hOh2IB/fnr7zxbs7qSBiTktXsspfa3nezrdNQ+iXdTRPrBit+2j67CrvKpBoRKe\nauY+7woPVDVJYjeAmy5Ly8zZUoGDlpRwKEtTu48108Cg3bISLhqycX18ZpZ2BA1N\nu+QU1Yn3RaGFSU/8DCJxtQ==\n-----END CERTIFICATE-----",
              "privateKey": "-----BEGIN PRIVATE KEY-----\nMIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQDoNm3IGT9YQliB\np1opKvIYr2atd9J317Mg4o39FaZL3xw5lXXxCMs/ZIUVVYAk7cOY5ZRrBmeDvtIx\nMfmxhsAFIM7R/99fCFvhFBXfdvI2lMIGIlqksG7YJB4xFB99FPW189Z7xSEjo1N9\n4pmybA6uEY76t/2MXpTHkRcNMYDAFoshBZ21VONXEBT+vj6mWydf8GeJ4U1pyRpm\nLxJrUNhYduyRkI5Kqyqdtj1fHPLXbPPgsgm4v+szkSRMY9s0P87jiFYJtus++Tkr\nLkX/lu3RQNC4us1HZO8tYsN2P8iOCPobXzkU7pvEjMxvfbLG7hNpTUCdmYeLVc/W\nx8DlNvqfAgMBAAECggEASSUH7J1DTkZLwb9Yz5nm+26YrbCOG9DWbFfguOUuZjzH\nk73oEj4eY4ACyacOf9NjJtC+MP4p8h5T8EoZKFnVN2hPrWdnUXR9GIduol7Byf6O\npUcB/VlT+QJbfkMj7g8BnMhLed4s46BpRsBvgHu4Hg2K15/IHoSWYcxqke3Ta+20\nQo/cJxfgr5lHtae6XXBnp/Mu+vMuQZxFAz5TR6bWPjwRqyok8Mk1JK9we0nEJm8U\nml66N9i0UerhvIyFXUMww+846J/HnyrRvz0j5EnYmcquDw3DMMXzpssYCzamueny\nMgiD6VujfT6pQ/ClQQHZf3+naDPA6dz4zeVOwxvquQKBgQD8FNvObw4TI2ZRNlIW\nv6By4JWAIut25NI6r6+UXMYHbKxP7bI97P1SQMV+kGdVxiKJ366BM18WlzOIurJy\nFKtE6QaggEZYN/duotA+jt3MygyXg2gu2wEpBzIwgxxRbR05pQLlsLtbSxKlpfZV\nk5trX0ZQ2eaUw0ITb43rsJ7VdwKBgQDr0oDeQv8FPCbsmgGeX0UWLP1aBeCJZNub\n15E5vXAy22fmsJTuAh7kvyyEHe4c/maug44VFEahn2H+6H2YGdzZTmGHzcxDpjSe\nIVQswxNCxpdgo/w5aR7M9Ewn5ollZotO0eIzj0MVaETW6UBqYX8oRgicpHC4KrKM\nGMabZJ1uGQKBgQCUZts4XpzUm4SCzw3ooouc1aZttyET74XsUr11BGD4wft3WqIS\nXtCLeeJKrkyHbIusy2h6W3nhXMZT+kVPb+ecO+tQ1fOTv+8EzQj3qzfcdh6PnCbb\nXscCFmBvuuAS97+6zfA0tKS4DCxAJMIugyV+QqqssntSnNjrhELyvBnl5QKBgQCk\nx+ioZiQQomGIfmyXH3cE8dbuaqDlIIabtNuTfx3BS7KkbcsDLJQtvq/6eXeC5vkV\nBHPpostf8CDnn8jy2U+KwMxBurn6o06tGBjbVkxFIsNwEeYSr7OH/0SftOVY53h8\nUQhAguCbOsqvaTlLnGjf2V/3JKhm597vKfjNaFbhMQKBgQCb7+WkDN9Pc3n1gbo3\nnZW7sTasIDj0MoZuMiM8J6R9EsD+hNJ/c5vpr4CeFaT4DTanhqLR6cCPR+Nqcl1T\neB4IKarz04svwizzFSG0XDCVmKO3nM4Zai0zkAIFSZxIXG9pxmucOmx8DPrvlX/c\nptO8AvBlmMvMiGlAMyDlKMUYSQ==\n-----END PRIVATE KEY-----"
            }
          }
        }
      }
    }
```

## 2.端到端加密

### 场景描述

F5并不做SSL卸载，在SSL解密后，连接后端真实服务器时再重新加密。这个功能适用于应用需要端到端加密，但又需要F5做一些反代或者7层安全的场景。

### 参考YAML

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-6
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
        "schemaVersion": "3.27.0",
        "remark": "End to end encryption",
        "cis-c-6": {
          "class": "Tenant",
          "cafe": {
            "class": "Application",
            "https_vs": {
              "class": "Service_HTTPS",
              "virtualAddresses": [
                "192.0.2.12"
              ],
              "virtualPort": 443,
              "pool": "coffee_pool",
              "serverTLS": "coffeetls",
              "clientTLS": "coffeetls-server",
              "redirect80": false
            },
            "coffee_pool": {
              "class": "Pool",
              "monitors": [
                "http"
              ],
              "members": [{
                "servicePort": 80,
                "serverAddresses": []
              }]
            },
            "coffeetls": {
              "class": "TLS_Server",
              "certificates": [{
                "certificate": "coffeecert"
              }]
            },
            "coffeecert": {
              "class": "Certificate",
              "remark": "coffee.example.com",
              "certificate": "-----BEGIN CERTIFICATE-----\nMIIDPDCCAiSgAwIBAgIEF6x2/TANBgkqhkiG9w0BAQsFADBgMQswCQYDVQQGEwJD\nTjELMAkGA1UECBMCWkoxCzAJBgNVBAcTAkhaMQswCQYDVQQKEwJGNTENMAsGA1UE\nCxMEVGVzdDEbMBkGA1UEAxMSY29mZmVlLmV4YW1wbGUuY29tMB4XDTIyMDgwMzA3\nMjM0MVoXDTMyMDczMTA3MjM0MVowYDELMAkGA1UEBhMCQ04xCzAJBgNVBAgTAlpK\nMQswCQYDVQQHEwJIWjELMAkGA1UEChMCRjUxDTALBgNVBAsTBFRlc3QxGzAZBgNV\nBAMTEmNvZmZlZS5leGFtcGxlLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC\nAQoCggEBAOg2bcgZP1hCWIGnWikq8hivZq130nfXsyDijf0VpkvfHDmVdfEIyz9k\nhRVVgCTtw5jllGsGZ4O+0jEx+bGGwAUgztH/318IW+EUFd928jaUwgYiWqSwbtgk\nHjEUH30U9bXz1nvFISOjU33imbJsDq4Rjvq3/YxelMeRFw0xgMAWiyEFnbVU41cQ\nFP6+PqZbJ1/wZ4nhTWnJGmYvEmtQ2Fh27JGQjkqrKp22PV8c8tds8+CyCbi/6zOR\nJExj2zQ/zuOIVgm26z75OSsuRf+W7dFA0Li6zUdk7y1iw3Y/yI4I+htfORTum8SM\nzG99ssbuE2lNQJ2Zh4tVz9bHwOU2+p8CAwEAATANBgkqhkiG9w0BAQsFAAOCAQEA\nQrzflgFiNs1pA4ou/1+q2o59/cw6ga5MXWbjVDCh34w9okpzNnEmPlBlvLiLykSV\n5H7u6pnNP1EUPdDe+Cleg0E2Om0pIwuvmBc8YT8AADfE+znGb/OUEQOZ4pGSbxeQ\nZX5/H5Ie4UszfcEPfNnBerMRX6OBy39RjIQBTvioSMCu4agfzY8eubQfDwBpRb/o\n+hOh2IB/fnr7zxbs7qSBiTktXsspfa3nezrdNQ+iXdTRPrBit+2j67CrvKpBoRKe\nauY+7woPVDVJYjeAmy5Ly8zZUoGDlpRwKEtTu48108Cg3bISLhqycX18ZpZ2BA1N\nu+QU1Yn3RaGFSU/8DCJxtQ==\n-----END CERTIFICATE-----",
              "privateKey": "-----BEGIN PRIVATE KEY-----\nMIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQDoNm3IGT9YQliB\np1opKvIYr2atd9J317Mg4o39FaZL3xw5lXXxCMs/ZIUVVYAk7cOY5ZRrBmeDvtIx\nMfmxhsAFIM7R/99fCFvhFBXfdvI2lMIGIlqksG7YJB4xFB99FPW189Z7xSEjo1N9\n4pmybA6uEY76t/2MXpTHkRcNMYDAFoshBZ21VONXEBT+vj6mWydf8GeJ4U1pyRpm\nLxJrUNhYduyRkI5Kqyqdtj1fHPLXbPPgsgm4v+szkSRMY9s0P87jiFYJtus++Tkr\nLkX/lu3RQNC4us1HZO8tYsN2P8iOCPobXzkU7pvEjMxvfbLG7hNpTUCdmYeLVc/W\nx8DlNvqfAgMBAAECggEASSUH7J1DTkZLwb9Yz5nm+26YrbCOG9DWbFfguOUuZjzH\nk73oEj4eY4ACyacOf9NjJtC+MP4p8h5T8EoZKFnVN2hPrWdnUXR9GIduol7Byf6O\npUcB/VlT+QJbfkMj7g8BnMhLed4s46BpRsBvgHu4Hg2K15/IHoSWYcxqke3Ta+20\nQo/cJxfgr5lHtae6XXBnp/Mu+vMuQZxFAz5TR6bWPjwRqyok8Mk1JK9we0nEJm8U\nml66N9i0UerhvIyFXUMww+846J/HnyrRvz0j5EnYmcquDw3DMMXzpssYCzamueny\nMgiD6VujfT6pQ/ClQQHZf3+naDPA6dz4zeVOwxvquQKBgQD8FNvObw4TI2ZRNlIW\nv6By4JWAIut25NI6r6+UXMYHbKxP7bI97P1SQMV+kGdVxiKJ366BM18WlzOIurJy\nFKtE6QaggEZYN/duotA+jt3MygyXg2gu2wEpBzIwgxxRbR05pQLlsLtbSxKlpfZV\nk5trX0ZQ2eaUw0ITb43rsJ7VdwKBgQDr0oDeQv8FPCbsmgGeX0UWLP1aBeCJZNub\n15E5vXAy22fmsJTuAh7kvyyEHe4c/maug44VFEahn2H+6H2YGdzZTmGHzcxDpjSe\nIVQswxNCxpdgo/w5aR7M9Ewn5ollZotO0eIzj0MVaETW6UBqYX8oRgicpHC4KrKM\nGMabZJ1uGQKBgQCUZts4XpzUm4SCzw3ooouc1aZttyET74XsUr11BGD4wft3WqIS\nXtCLeeJKrkyHbIusy2h6W3nhXMZT+kVPb+ecO+tQ1fOTv+8EzQj3qzfcdh6PnCbb\nXscCFmBvuuAS97+6zfA0tKS4DCxAJMIugyV+QqqssntSnNjrhELyvBnl5QKBgQCk\nx+ioZiQQomGIfmyXH3cE8dbuaqDlIIabtNuTfx3BS7KkbcsDLJQtvq/6eXeC5vkV\nBHPpostf8CDnn8jy2U+KwMxBurn6o06tGBjbVkxFIsNwEeYSr7OH/0SftOVY53h8\nUQhAguCbOsqvaTlLnGjf2V/3JKhm597vKfjNaFbhMQKBgQCb7+WkDN9Pc3n1gbo3\nnZW7sTasIDj0MoZuMiM8J6R9EsD+hNJ/c5vpr4CeFaT4DTanhqLR6cCPR+Nqcl1T\neB4IKarz04svwizzFSG0XDCVmKO3nM4Zai0zkAIFSZxIXG9pxmucOmx8DPrvlX/c\nptO8AvBlmMvMiGlAMyDlKMUYSQ==\n-----END PRIVATE KEY-----"
            },
            "coffeetls-server" :{
              "class": "TLS_Client"
            }
          }
        }
      }
    }
```

在F5的配置中，这个功能是通过vs挂载server-ssl profile来实现的，**但请注意**，在AS3声明中，对于Client和Server的定义和F5配置中的client-ssl和server-ssl是相反的。

在上述AS3声明中，`"serverTLS"`对应的是F5上的client-ssl profile，`"clientTLS"`对应的是F5上的server-ssl profile。

### 部署结果

```
ltm virtual cafe/https_vs {
    creation-time 2022-08-19:16:26:14
    description cafe
    destination 192.0.2.12:https
    ip-protocol tcp
    last-modified-time 2022-08-19:16:26:14
    mask 255.255.255.255
    partition cis-c-6
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool cafe/coffee_pool
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
        cafe/coffeetls {
            context clientside
        }
        cafe/coffeetls-server {
            context serverside
        }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 54
}


ltm profile server-ssl cafe/coffeetls-server {
    alert-timeout indefinite
    allow-expired-crl disabled
    app-service none
    authenticate once
    authenticate-name none
    c3d-cert-extension-includes { basic-constraints extended-key-usage key-usage subject-alternative-name }
    c3d-cert-lifespan 24
    ca-file /Common/ca-bundle.crt
    cache-timeout 3600
    cipher-group none
    ciphers DEFAULT
    expire-cert-response-control drop
    peer-cert-mode ignore
    renegotiation enabled
    retain-certificate true
    server-name none
    session-ticket disabled
    ssl-c3d disabled
    ssl-forward-proxy disabled
    ssl-forward-proxy-bypass disabled
    untrusted-cert-response-control drop
}
```

## 3.调用已有证书

### 场景描述

上面的范例中我们都是通过AS3创建证书的和密钥，但实际使用当中我们可能会多个应用共用一张证书，例如泛域名证书，这时候我们只需要创建证书和密钥一次即可。下面的范例我们会使用/Common分区下已有的证书和密钥。

### 参考YAML

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-6
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
        "schemaVersion": "3.27.0",
        "remark": "Use existing cert",
        "cis-c-6": {
          "class": "Tenant",
          "cafe": {
            "class": "Application",
            "https_vs": {
              "class": "Service_HTTPS",
              "virtualAddresses": [
                "192.0.2.13"
              ],
              "virtualPort": 443,
              "pool": "coffee_pool",
              "serverTLS": "cafetls",
              "redirect80": false
            },
            "coffee_pool": {
              "class": "Pool",
              "monitors": [
                "http"
              ],
              "members": [{
                "servicePort": 80,
                "serverAddresses": []
              }]
            },
            "cafetls": {
              "class": "TLS_Server",
              "certificates": [{
                "certificate": "cafecert"
              }]
            },
            "cafecert": {
              "class": "Certificate",
              "remark": "cafe.example.com",
              "certificate": {
                "bigip": "/Common/cafe.example.com"
              },
              "privateKey": {
                "bigip": "/Common/cafe.example.com"
              }
            }
          }
        }
      }
    }
```

### 部署结果

```
ltm virtual cafe/https_vs {
    creation-time 2022-08-19:16:32:53
    description cafe
    destination 192.0.2.13:https
    ip-protocol tcp
    last-modified-time 2022-08-19:16:32:53
    mask 255.255.255.255
    partition cis-c-6
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool cafe/coffee_pool
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
        cafe/cafetls {
            context clientside
        }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 55
}

ltm profile client-ssl cafe/cafetls {
    alert-timeout indefinite
    allow-expired-crl disabled
    app-service none
    authenticate once
    c3d-drop-unknown-ocsp-status drop
    cache-timeout 3600
    cert /Common/cafe.example.com
    cert-key-chain {
        cafe.example {
            cert /Common/cafe.example.com
            key /Common/cafe.example.com
        }
    }
    cert-lookup-by-ipaddr-port disabled
    cipher-group none
    ciphers DEFAULT
    inherit-ca-certkeychain true
    inherit-certkeychain false
    key /Common/cafe.example.com
    ocsp-stapling disabled
    peer-cert-mode ignore
    renegotiation enabled
    retain-certificate true
    server-name none
    sni-default true
    sni-require false
    ssl-c3d disabled
    ssl-forward-proxy disabled
    ssl-forward-proxy-bypass disabled
}
```

## 4.根据SNI匹配证书

### 场景描述

在一个虚拟服务上绑定多张证书，F5根据SNI来自动匹配证书。

### 参考YAML

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-6
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
        "schemaVersion": "3.27.0",
        "remark": "One https vs with multiple SSL cert",
        "cis-c-6": {
          "class": "Tenant",
          "cafe": {
            "class": "Application",
            "https_vs": {
              "class": "Service_HTTPS",
              "virtualAddresses": [
                "192.0.2.14"
              ],
              "virtualPort": 443,
              "pool": "coffee_pool",
              "serverTLS": "cafetls",
              "redirect80": false
            },
            "coffee_pool": {
              "class": "Pool",
              "monitors": [
                "http"
              ],kube
              "members": [{
                "servicePort": 80,
                "serverAddresses": []
              }]
            },
            "cafetls": {
              "class": "TLS_Server",
              "certificates": [
                {
                  "matchToSNI": "coffee.example.com",
                  "certificate": "coffeecert"
                },
                {
                  "matchToSNI": "tea.example.com",
                  "certificate": "teacert"
                }
              ]
            },
            "coffeecert": {
              "class": "Certificate",
              "remark": "coffee.example.com",
              "certificate": "-----BEGIN CERTIFICATE-----\nMIIDPDCCAiSgAwIBAgIEF6x2/TANBgkqhkiG9w0BAQsFADBgMQswCQYDVQQGEwJD\nTjELMAkGA1UECBMCWkoxCzAJBgNVBAcTAkhaMQswCQYDVQQKEwJGNTENMAsGA1UE\nCxMEVGVzdDEbMBkGA1UEAxMSY29mZmVlLmV4YW1wbGUuY29tMB4XDTIyMDgwMzA3\nMjM0MVoXDTMyMDczMTA3MjM0MVowYDELMAkGA1UEBhMCQ04xCzAJBgNVBAgTAlpK\nMQswCQYDVQQHEwJIWjELMAkGA1UEChMCRjUxDTALBgNVBAsTBFRlc3QxGzAZBgNV\nBAMTEmNvZmZlZS5leGFtcGxlLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC\nAQoCggEBAOg2bcgZP1hCWIGnWikq8hivZq130nfXsyDijf0VpkvfHDmVdfEIyz9k\nhRVVgCTtw5jllGsGZ4O+0jEx+bGGwAUgztH/318IW+EUFd928jaUwgYiWqSwbtgk\nHjEUH30U9bXz1nvFISOjU33imbJsDq4Rjvq3/YxelMeRFw0xgMAWiyEFnbVU41cQ\nFP6+PqZbJ1/wZ4nhTWnJGmYvEmtQ2Fh27JGQjkqrKp22PV8c8tds8+CyCbi/6zOR\nJExj2zQ/zuOIVgm26z75OSsuRf+W7dFA0Li6zUdk7y1iw3Y/yI4I+htfORTum8SM\nzG99ssbuE2lNQJ2Zh4tVz9bHwOU2+p8CAwEAATANBgkqhkiG9w0BAQsFAAOCAQEA\nQrzflgFiNs1pA4ou/1+q2o59/cw6ga5MXWbjVDCh34w9okpzNnEmPlBlvLiLykSV\n5H7u6pnNP1EUPdDe+Cleg0E2Om0pIwuvmBc8YT8AADfE+znGb/OUEQOZ4pGSbxeQ\nZX5/H5Ie4UszfcEPfNnBerMRX6OBy39RjIQBTvioSMCu4agfzY8eubQfDwBpRb/o\n+hOh2IB/fnr7zxbs7qSBiTktXsspfa3nezrdNQ+iXdTRPrBit+2j67CrvKpBoRKe\nauY+7woPVDVJYjeAmy5Ly8zZUoGDlpRwKEtTu48108Cg3bISLhqycX18ZpZ2BA1N\nu+QU1Yn3RaGFSU/8DCJxtQ==\n-----END CERTIFICATE-----",
              "privateKey": "-----BEGIN PRIVATE KEY-----\nMIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQDoNm3IGT9YQliB\np1opKvIYr2atd9J317Mg4o39FaZL3xw5lXXxCMs/ZIUVVYAk7cOY5ZRrBmeDvtIx\nMfmxhsAFIM7R/99fCFvhFBXfdvI2lMIGIlqksG7YJB4xFB99FPW189Z7xSEjo1N9\n4pmybA6uEY76t/2MXpTHkRcNMYDAFoshBZ21VONXEBT+vj6mWydf8GeJ4U1pyRpm\nLxJrUNhYduyRkI5Kqyqdtj1fHPLXbPPgsgm4v+szkSRMY9s0P87jiFYJtus++Tkr\nLkX/lu3RQNC4us1HZO8tYsN2P8iOCPobXzkU7pvEjMxvfbLG7hNpTUCdmYeLVc/W\nx8DlNvqfAgMBAAECggEASSUH7J1DTkZLwb9Yz5nm+26YrbCOG9DWbFfguOUuZjzH\nk73oEj4eY4ACyacOf9NjJtC+MP4p8h5T8EoZKFnVN2hPrWdnUXR9GIduol7Byf6O\npUcB/VlT+QJbfkMj7g8BnMhLed4s46BpRsBvgHu4Hg2K15/IHoSWYcxqke3Ta+20\nQo/cJxfgr5lHtae6XXBnp/Mu+vMuQZxFAz5TR6bWPjwRqyok8Mk1JK9we0nEJm8U\nml66N9i0UerhvIyFXUMww+846J/HnyrRvz0j5EnYmcquDw3DMMXzpssYCzamueny\nMgiD6VujfT6pQ/ClQQHZf3+naDPA6dz4zeVOwxvquQKBgQD8FNvObw4TI2ZRNlIW\nv6By4JWAIut25NI6r6+UXMYHbKxP7bI97P1SQMV+kGdVxiKJ366BM18WlzOIurJy\nFKtE6QaggEZYN/duotA+jt3MygyXg2gu2wEpBzIwgxxRbR05pQLlsLtbSxKlpfZV\nk5trX0ZQ2eaUw0ITb43rsJ7VdwKBgQDr0oDeQv8FPCbsmgGeX0UWLP1aBeCJZNub\n15E5vXAy22fmsJTuAh7kvyyEHe4c/maug44VFEahn2H+6H2YGdzZTmGHzcxDpjSe\nIVQswxNCxpdgo/w5aR7M9Ewn5ollZotO0eIzj0MVaETW6UBqYX8oRgicpHC4KrKM\nGMabZJ1uGQKBgQCUZts4XpzUm4SCzw3ooouc1aZttyET74XsUr11BGD4wft3WqIS\nXtCLeeJKrkyHbIusy2h6W3nhXMZT+kVPb+ecO+tQ1fOTv+8EzQj3qzfcdh6PnCbb\nXscCFmBvuuAS97+6zfA0tKS4DCxAJMIugyV+QqqssntSnNjrhELyvBnl5QKBgQCk\nx+ioZiQQomGIfmyXH3cE8dbuaqDlIIabtNuTfx3BS7KkbcsDLJQtvq/6eXeC5vkV\nBHPpostf8CDnn8jy2U+KwMxBurn6o06tGBjbVkxFIsNwEeYSr7OH/0SftOVY53h8\nUQhAguCbOsqvaTlLnGjf2V/3JKhm597vKfjNaFbhMQKBgQCb7+WkDN9Pc3n1gbo3\nnZW7sTasIDj0MoZuMiM8J6R9EsD+hNJ/c5vpr4CeFaT4DTanhqLR6cCPR+Nqcl1T\neB4IKarz04svwizzFSG0XDCVmKO3nM4Zai0zkAIFSZxIXG9pxmucOmx8DPrvlX/c\nptO8AvBlmMvMiGlAMyDlKMUYSQ==\n-----END PRIVATE KEY-----"
            },
            "teacert": {
              "class": "Certificate",
              "remark": "tea.example.com",
              "certificate": "-----BEGIN CERTIFICATE-----\nMIIDNjCCAh6gAwIBAgIEF6yK6DANBgkqhkiG9w0BAQsFADBdMQswCQYDVQQGEwJD\nTjELMAkGA1UECBMCWkoxCzAJBgNVBAcTAkhaMQswCQYDVQQKEwJGNTENMAsGA1UE\nCxMEVGVzdDEYMBYGA1UEAxMPdGVhLmV4YW1wbGUuY29tMB4XDTIyMDgwMzA4NDg0\nMFoXDTMyMDczMTA4NDg0MFowXTELMAkGA1UEBhMCQ04xCzAJBgNVBAgTAlpKMQsw\nCQYDVQQHEwJIWjELMAkGA1UEChMCRjUxDTALBgNVBAsTBFRlc3QxGDAWBgNVBAMT\nD3RlYS5leGFtcGxlLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB\nAOxIFhA7yv1p4gzcbWR4x07f0zhKH8VfyWTRnw7QjZFUjNLbEa7bOhQt53V9oCSR\nol/8tY22CmBSAtlmTt5FX5H6W6naU8EAZUmwHO3eXjp/a1HUk0fj4D++dJ8KEF0b\nT2fxq4H+jYoTDE/Nr4u9TExuZfU/tcMsF3PrgIfkjydjW7peyJoxhvLt4tyk8z3a\nkfSgs6mjlBuc5/2Y1tIStRpJ7gl+eKueDHp4ogPnx5qcg+GAM1pYiv17wKVJzcHW\nDor0B4xVi3CQWlUlTwW1TX+pwL0JkoOOPfbNYo67G43DmG8m3WNG+FH+aIGXhyyp\ncy4zbum4d1HZI/zp+k7qd6ECAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAlfg7nBJL\nDQWDGjczOjZjOcHrM7jbfuadmZQAgmASGRLMaHNuj8RIG+Ro+znM6QYNB7S8E4ZF\ntjfvamSoFSB5+YoOwFZlyWyiHhW1L22HB7G2JabtBhexWURNfsjrwkROGpRBqjGz\neuq4x/kkRYkOGhWGyXmJQmPo2FjO2g4R1zIRGEMeij0xpPuYIZsA6BOuyYiJPvi0\nFBSsc+udZqOz9/MzcrU2mmIX0N42XudhWbiXNe6izPvBsrztJoZA8gaHn99JgZcZ\ngKLYKl6FW4aN5mh2XGwuS+smbD37vr4J3faZg+18KN+H6g8mbF/ArPxKP9fiM9u9\npKTWOTrCU0hJkw==\n-----END CERTIFICATE-----",
              "privateKey": "-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDsSBYQO8r9aeIM\n3G1keMdO39M4Sh/FX8lk0Z8O0I2RVIzS2xGu2zoULed1faAkkaJf/LWNtgpgUgLZ\nZk7eRV+R+lup2lPBAGVJsBzt3l46f2tR1JNH4+A/vnSfChBdG09n8auB/o2KEwxP\nza+LvUxMbmX1P7XDLBdz64CH5I8nY1u6XsiaMYby7eLcpPM92pH0oLOpo5QbnOf9\nmNbSErUaSe4Jfnirngx6eKID58eanIPhgDNaWIr9e8ClSc3B1g6K9AeMVYtwkFpV\nJU8FtU1/qcC9CZKDjj32zWKOuxuNw5hvJt1jRvhR/miBl4csqXMuM27puHdR2SP8\n6fpO6nehAgMBAAECggEAYScGw2gCgA5AXy9nX2917A2GKNf5lktbYLP8ZbgE7aPJ\nP43KCI6lo9R4HkwoQ8EJ9dPPxtP6Ej7GYyN4/FWkBT7e38kgtPP3scPTMU9EiWMI\n+p2gbWfaNfuWsioOLmpjTQcGkS3cftB0OIAHVTrhm2+tRpkKoJSJlCVaNQYagoPC\nzhns8KJceDnyRikRfH0RbPPGsLLohnRYPDEs6J0yCXvUhKxsFKXq/mL0licu9KZN\nqUwX2/TmT8iDnoC+TBkYJk055llBv16l5Hx7egYD9uNEFk76SInC2MCHpl1nlp43\nEp5uRUsaCjS+5tpqugQPelhFQC9v7l4t4x+3xH/XPQKBgQD7riYMLVCNnSH3zRH8\nYOwwFqGVSngTZw1hlTd06HEzUpntlHaoO3Z9pH2XWgaPF+2e2Hngm7EMNe5qjZau\nlyOqb85UCxuAs0wLlLD/Q4sWOGmEPrkIHmKXHMZjQMjtqCkKr//zvE0W9ADo2MLm\nawTxj45gTZxGXvuTEnMMkaflKwKBgQDwVkcXIJImgK+JXEjM1rGeypONVCnAqo7p\nFJq2/QAraGvlNLv1QBzE6s4uqwWfzCecICQOoNrLG0dE3mHIlYgVqOK5Li9CJcsB\n6dfEzgQyT4hotghCC/udjkAvKskXd4ShXbRSvF3p3elZDgXfAbQ5bxIL4QIOJF6f\nd5os2LqIYwKBgQC3thDzxogMNuy6kyhTzvPYzkw4S1mG4Cw2VNNcNOecjOjrMPnE\nJ1OAtvct6XrsLI04689bEoqT3TIg+SVKX+ya1m4HjuwOb9JMiccBLW5zU85Bx/8M\nXBGfOFPf00RXpe3/bSUp5wNmg8m+LatmwiujoCRPS5eNDnwYiNkODaw+bQKBgG5Z\nfvysdM5+6ZotKDP9I8LgCo2qph0TctisIDmCwvArWtb7to1t6Ye0tASTe9qaN1ml\nHEknLC5zkO6bGNSra7deOvOBtCswBR0UzIBNg3nCMMS7R+FjdR0rcmb1wy0mMFyT\nFLekS46U2I6ONL3nH2P7jpKrtnDd3CBmHwEWZdc3AoGBAM6ssZsvW47BGBbYNu7x\nwEE0ehL/lesNQK9qEvBbtQ3w8ta5M9QHNu0vg05A6Pv+fLhSA+RuutAnF+bqBMpA\nRM3JjvSRiMO51nISmVNl9Hga3vVvlUc7FDomv+KHc/RE6FqRdDRRvflErKb4sWDa\nV4BS1wQmnyROovJMDN0A/mVi\n-----END PRIVATE KEY-----"
            }
          }
        }
      }
    }
```

### 部署结果

```
ltm virtual cafe/https_vs {
    creation-time 2022-08-19:16:38:34
    description cafe
    destination 192.0.2.14:https
    ip-protocol tcp
    last-modified-time 2022-08-19:16:38:34
    mask 255.255.255.255
    partition cis-c-6
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool cafe/coffee_pool
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
        cafe/cafetls {
            context clientside
        }
        cafe/cafetls-1- {
            context clientside
        }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 56
}

ltm profile client-ssl cafe/cafetls {
    alert-timeout indefinite
    allow-expired-crl disabled
    app-service none
    authenticate once
    c3d-drop-unknown-ocsp-status drop
    cache-timeout 3600
    cert cafe/coffeecert.crt
    cert-key-chain {
        coffeecert {
            cert cafe/coffeecert.crt
            key cafe/coffeecert.key
        }
    }
    cert-lookup-by-ipaddr-port disabled
    cipher-group none
    ciphers DEFAULT
    inherit-ca-certkeychain true
    inherit-certkeychain false
    key cafe/coffeecert.key
    ocsp-stapling disabled
    peer-cert-mode ignore
    renegotiation enabled
    retain-certificate true
    server-name coffee.example.com
    sni-default true
    sni-require false
    ssl-c3d disabled
    ssl-forward-proxy disabled
    ssl-forward-proxy-bypass disabled
}

ltm profile client-ssl cafe/cafetls-1- {
    alert-timeout indefinite
    allow-expired-crl disabled
    app-service none
    authenticate once
    c3d-drop-unknown-ocsp-status drop
    cache-timeout 3600
    cert cafe/teacert.crt
    cert-key-chain {
        teacert {
            cert cafe/teacert.crt
            key cafe/teacert.key
        }
    }
    cert-lookup-by-ipaddr-port disabled
    cipher-group none
    ciphers DEFAULT
    inherit-ca-certkeychain true
    inherit-certkeychain false
    key cafe/teacert.key
    ocsp-stapling disabled
    peer-cert-mode ignore
    renegotiation enabled
    retain-certificate true
    server-name tea.example.com
    sni-default false
    sni-require false
    ssl-c3d disabled
    ssl-forward-proxy disabled
    ssl-forward-proxy-bypass disabled
}
```

## 5.双向认证

### 场景描述

除了出示服务器证书外，应用还需要对客户端进行证书认证，需要在F5上配置CA证书。

### 参考YAML

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cis-c-6
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
        "schemaVersion": "3.27.0",
        "remark": "HTTPS with bidirection auth",
        "cis-c-6": {
          "class": "Tenant",
          "cafe": {
            "class": "Application",
            "https_vs": {
              "class": "Service_HTTPS",
              "virtualAddresses": [
                "192.0.2.15"
              ],
              "virtualPort": 443,
              "pool": "coffee_pool",
              "serverTLS": "coffeetls",
              "redirect80": false
            },
            "coffee_pool": {
              "class": "Pool",
              "monitors": [
                "http"
              ],
              "members": [{
                "servicePort": 80,
                "serverAddresses": []
              }]
            },
            "coffeetls": {
              "class": "TLS_Server",
              "certificates": [{
                "certificate": "coffeecert"
              }],
              "authenticationMode": "require",
              "authenticationFrequency": "every-time",
              "authenticationTrustCA": "coffeeca"
            },
            "coffeecert": {
              "class": "Certificate",
              "remark": "coffee.example.com",
              "certificate": {
                "bigip": "/Common/default.crt"
              },
              "privateKey": {
                "bigip": "/Common/default.key"
              }
            },
            "coffeeca": {
              "class": "CA_Bundle",
              "bundle": {
                "bigip": "/Common/ca-bundle.crt"
              }
            }
          }
        }
      }
    }
```

### 部署结果

```
ltm virtual cafe/https_vs {
    creation-time 2022-08-19:16:47:53
    description cafe
    destination 192.0.2.15:https
    ip-protocol tcp
    last-modified-time 2022-08-19:16:47:53
    mask 255.255.255.255
    partition cis-c-6
    persist {
        /Common/cookie {
            default yes
        }
    }
    pool cafe/coffee_pool
    profiles {
        /Common/f5-tcp-progressive { }
        /Common/http { }
        cafe/coffeetls {
            context clientside
        }
    }
    serverssl-use-sni disabled
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 57
}

ltm profile client-ssl cafe/coffeetls {
    alert-timeout indefinite
    allow-expired-crl disabled
    app-service none
    authenticate always
    c3d-drop-unknown-ocsp-status drop
    ca-file /Common/ca-bundle.crt
    cache-timeout 3600
    cert /Common/default.crt
    cert-key-chain {
        default {
            cert /Common/default.crt
            key /Common/default.key
        }
    }
    cert-lookup-by-ipaddr-port disabled
    cipher-group none
    ciphers DEFAULT
    inherit-ca-certkeychain true
    inherit-certkeychain false
    key /Common/default.key
    ocsp-stapling disabled
    peer-cert-mode require
    renegotiation enabled
    retain-certificate true
    server-name none
    sni-default true
    sni-require false
    ssl-c3d disabled
    ssl-forward-proxy disabled
    ssl-forward-proxy-bypass disabled
}
```
