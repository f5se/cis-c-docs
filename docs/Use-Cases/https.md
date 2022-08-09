# 1.标准HTTPS业务发布范例

## 场景描述

最常用的HTTPS业务发布，前端使用HTTPS，导入并配置服务器证书和密钥，后端使用HTTP，实现SSL Offloading。

## 参考YAML

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

## 部署结果

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

# 2.端到端加密

## 场景描述

F5并不做SSL卸载，在SSL解密后，连接后端真实服务器时再重新加密。这个功能适用于应用需要端到端加密，但又需要F5做一些反代或者7层安全的场景。

## 参考YAML

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
              "clientTLS": [
                {
                  "bigip": "/Common/serverssl"
                }
              ],
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

在F5的配置中，这个功能是通过vs挂载server-ssl profile来实现的，**但请注意**，在AS3声明中，对于Client和Server的定义和F5配置中的client-ssl和server-ssl是相反的。

在上述AS3声明中，`"serverTLS"`对应的是F5上的client-ssl profile，`"clientTLS"`对应的是F5上的server-ssl profile。

## 部署结果

```
ltm virtual cafe/https_vs {
    creation-time 2022-08-09:11:15:47
    description cafe
    destination 192.0.2.12:https
    ip-protocol tcp
    last-modified-time 2022-08-09:11:15:47
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
        /Common/serverssl {
            context serverside
        }
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
    vs-index 50
}
```

