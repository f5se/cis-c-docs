# 版本发布

## Release: **2.9.1-20220930**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220930](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增加对k8s [IPv6环境及v4-v6混合部署环境](../Use-Cases/ipv6.md)支持
* 增加[与CIS共存](../Use-Cases/cis-and-cis-c.md)能力支持，归置arp于独立partition
* 增加针对BIGIP连接异常如40x 50x，conn-refused等的容错能力
* 增加用户配置持久化支持
* 增强AS3数据校验能力（针对AS3报文>100kB的场景）
* 修复完善Prometheus监控[metrics](../quick-start/prometheus.md)，增加函数级性能监控
* 其他功能修复和性能检测调优

## Release: **2.9.1-20220831**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220831](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增加并完善hub mode支持
* 增加TLS SNI、CA_Bundle及TLS_Client支持
* 增加并完善AS3中snat:self及Service_HTTPS:redirect80支持
* 增加ltm/snat-translation 类型支持
* 增加日志级别 “trace”
* 优化网络创建、持久化、node删除，以实现性能调优
* 修复iRule下发时特殊字符 < > & 处理异常
* 修复发布镜像的安全问题CVE-2022-2097
* Virtual Server 的 Connection Mirroring 打开或关闭
* 其他已知问题修复

## Release: **2.9.1-20220804**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220804](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

#### 部署与环境支持
* 支持集群内和集群外运行两种模式
* 支持k8s集群业务配置方式：deployment+service，应用交付（应用对外发布）配置方式：configmap([F5 AS3](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/)作为数据主体)
* 支持k8s CNI类型：Flannel vxlan或其它underlay模式CNI（如Calico路由模式，Kube-ovn underlay模式等），配置细节可参考[这里](/Architecture/parameters/)
* 支持广泛的k8s版本，如您发现k8s的版本支持问题，请[联系我们](/Support-and-contact/)
* 暂不支持Openshift
* 支持x86 CPU。 国产或ARM CPU支持请与我们[联系](/Support-and-contact/)

#### 功能与行为支持
* 支持通过`namespace`启动参数指定业务发布范围
* 支持使用`namespace-label`启动参数标记业务发布范围，实现指定范围的动态性
* 支持hub-mode方式，以便业务和应用交付配置可以在不同的namespace中分别进行，实现不同角色团队的工作解耦
* 支持服务绑定保护，避免误操作带来的服务覆盖异常
* 支持默认服务端口绑定：当configmap应用配置中无端口指定时，自动选择服务端口
* 支持增量处理，应用交付配置下发速度快
* 支持BIG-IP上扩展应用交付能力包括以下配置对象（及其子参数配置项），支持用户充分发挥BIG-IP能力。更多场景可参考[这里](/Use-Cases/http/)：
  ```
  * virtual
  * virtual-address
  * snatpool
  * pool/member/node
  * monitor：gateway_icmp tcp udp http
  * profile: tcp ftp udp http clientssl oneconnect
  * persist: cookie source_addr
  * iRules
  ```
* 支持IPv64转换,即对外发布IPv6访问IP，集群内使用IPv4。亦支持集群内IPv6

#### 运维与升级支持
* 兼容用户当前在CIS中已经发布的configmap yaml格式，用户在CIS-C中无需修改应用发布yaml
* 支持从CIS升级到CIS-C，需要借助kic-tool工具完成升级迁移
* 支持日志级别输出、和任务跟踪（request_id）
* 支持与Prometheus集成，采集控制器性能数据
* 在有效管理下，F5 CIS-C可以与F5 CIS同时运行，实现更灵活丰富的场景应对
