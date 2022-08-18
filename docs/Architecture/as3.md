# AS3内容配置指引

用户的业务配置通过configmap方式交付给CIS-C完成业务下发，configmap中的内容格式为[AS3](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/) JSON字符串，这一点与CIS configmap模式一致。

与CIS不同的是，CIS-C为追求下发性能，在程序内部实现了AS3格式的解析，然后通过iControlRest方式完成配置下发。CIS则是依靠运行于BIG-IP上的AS3 iApp程序完成配置解析、下发。

随着CIS-C项目的推进和客户需求的增加，CIS-C对AS3格式的解析能力和范围会不断完善、丰富。

目前，CIS-C支持的AS3类型和AS3字段，如下表所示。

*如需了解更多关于“AS3类型”和各个“AS3字段”的含义，请参考：AS3官网[Schema定义](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/refguide/schema-reference-byclass.html)。*

| AS3类型      | AS3字段集合 |
| ----------- | ----------- |
|                  AS3 |['declaration']                                                       |
|                  ADC |[]                                                                    |
|          Application |['template']                                                          |
|               Tenant |[]                                                                    |
|         Service_HTTP |['virtualAddresses', 'virtualPort', 'profileHTTP', 'persistenceMethods', 'iRules', 'pool', 'serviceDownImmediateAction', 'snat', 'profileMultiplex', 'profileTCP']|
|          Service_TCP |['virtualPort', 'virtualAddresses', 'profileTCP', 'pool', 'snat', 'serviceDownImmediateAction', 'profileFTP']|
|        Service_HTTPS |['virtualAddresses', 'redirect80', 'serverTLS', 'pool', 'virtualPort', 'clientTLS', 'profileHTTP', 'profileMultiplex', 'iRules']|
|      Service_Address |['virtualAddress', 'arpEnabled']                                      |
|          Service_UDP |['virtualPort', 'virtualAddresses', 'profileUDP', 'pool', 'snat']     |
|           Service_L4 |['virtualAddresses', 'virtualPort', 'layer4', 'persistenceMethods', 'profileL4', 'pool']|
|         HTTP_Profile |['xForwardedFor', 'insertHeader', 'hstsPeriod', 'headerErase', 'fallbackRedirect', 'fallbackStatusCodes']|
|    Multiplex_Profile |['sourceMask', 'maxConnectionReuse', 'maxConnections', 'maxConnectionAge', 'idleTimeoutOverride', 'connectionLimitEnforcement']|
|          TCP_Profile |['idleTimeout']                                                       |
|          UDP_Profile |['idleTimeout', 'datagramLoadBalancing']                              |
|          FTP_Profile |['remark', 'port', 'ftpsMode', 'enforceTlsSessionReuseEnabled', 'activeModeEnabled', 'securityEnabled', 'translateExtendedEnabled', 'inheritParentProfileEnabled']|
|                 Pool |['monitors', 'minimumMonitors', 'members', 'minimumMembersActive', 'loadBalancingMode']|
|                iRule |['remark', 'iRule']                                                   |
|              Persist |['persistenceMethod', 'encrypt', 'cookieMethod', 'passphrase', 'hashAlgorithm', 'addressMask', 'duration']|
|              Monitor |['monitorType', 'send', 'interval', 'timeout', 'receive']             |
|            SNAT_Pool |['snatAddresses']                                                     |
|           TLS_Server |['certificates', 'authenticationMode', 'authenticationFrequency', 'authenticationTrustCA']|
|          Certificate |['remark', 'certificate', 'privateKey', 'passphrase', 'chainCA']      |
|            CA_Bundle |['bundle']                                                            |
|           TLS_Client |['trustCA', 'sendSNI', 'ciphers', 'serverName', 'validateCertificate', 'ignoreExpired', 'ignoreUntrusted', 'sessionTickets', 'clientCertificate']|

<div style="display:none">
参考<代码库> tests/funcs/test.automation/collect_types.py 自动生成该表格。
</div>