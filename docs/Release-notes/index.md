# 版本发布

*Note:*

*Release Note中的commit链接为内部gitlab链接，外部并不能直接访问。*

*如有源码需求，可联系F5支持人员，获取源码zip文件。*

## Release: **2.14.9-20230901**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.9-20230901](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 优化已下发资源持久化性能。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/3d77a73e45d1117a6f20ddc2b4d9a5e8b82d1622)
* 修复Flannel VXLAN模式下的ARP创建、删除异常。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/ee0f269c3a2468b306889bb8917f837184186653)
* 增强DevOps：/dumps中增加启动时间标记[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/d4d206a30a77211ba156aa1cb1c4dfd187a99196)和事件队列状态[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/3d77a73e45d1117a6f20ddc2b4d9a5e8b82d1622)

## Release: **2.14.8-20230811**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.8-20230811](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 修复因为Service Label变化导致的pool member下发异常。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/e533cf6e805d257598e8a23899fc143840341dfb)

## Release: **2.14.7-20230808**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.7-20230808](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增加prestop回调功能中双栈情况下的参数检查，[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/4a30f3d4154d72ca521e3af4d1ba1aba0bd51448)
* 优化调整代码结构，[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/b2f91276c33eac1a6949286194abd4bf87fd0825)

## Release: **2.14.6-20230728**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.6-20230728](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 优化preStop调用过程，改从BIG-IP侧发现member为从k8s侧发现，以减少调用时间。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/67a99a3f738d712df23b0991877de2b7da515a6e)
* 优化请求处理队列，合并同类请求以减少重复BIG-IP调用次数。commits [1](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/35a7d00b0ec4363c848177bb2a53f9ac0011d787), [2](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/5c192eafbf1c16b8c35f48e6fe301b8a5594df76)
* 优化请求处理队列中对node的处理，删除重复的请求事件。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/f975718733416a63fc6198c13f128fdc6d23e9e8)


## Release: **2.14.5-20230726**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.5-20230726](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 优化preStop调用机制，引入500异常时的重试能力。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/b5ba24992bf447a17effdaffee64957877136599)
* 规范代码结构，删除不必要注释。commits [1](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/f072216b88626cdeb630032a7726b9366b821e7a), [2](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/32405ea6d2798dbe788c551dca4070c8fa317a32), [3](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/e441ced0d09da237abfddf2dbd1caccc746e2fcc)

## Release: **2.14.4-20230724**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.4-20230724](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增加dry-run模式用于加速开机启动过程。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/2bd1d1d0598348e231b62d914237468f8a24fc4b)

## Release: **2.14.3-20230721**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.3-20230721](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 性能优化。
  * 优化当业务量巨大时持久化策略。 commits [1](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/1b58e6f133c9129c16e44fbe22ae510d2dce4abd), [2](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/a34fe0b48d09ff79b42007ccf7036c2348b16a2e)
  * 修改启动时逐个加载data-group为一次性加载所有，提高启动速度。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/9006a557c68e568fb33541670c7e5ac22b515121)
  * 修改业务和应用的串行化配置为周期性任务。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/8c8f86a12a67b3dc88478fc198a3f21802b5159c)
* 修复多个service共享同一个deployment导致BIG-IP node共享时无法删除业务问题。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/fb03f8776edb7334ae1a18157ecd549c7f979815)
* 修复统一资源并发操作（创建随即马上删除，CIS-C还未处理完成创建）时的清除失败问题。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/a34fe0b48d09ff79b42007ccf7036c2348b16a2e)
* 修改prestop异步调用为同步阻塞调用，避免事件队列过长引起的member状态更新延后。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/064f79a10f1fcc7f9293bdcdf2ee47f8e36e7197)
* 代码规范性调整。commits [1](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/88abc8d379a5fe8c11fc95a4726a3ac119557c79), [2](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/de16603d40c45ed8bdb8554e47a4e4a327159618)

## Release: **2.14.2-20230713**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.2-20230713](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增加参数--pool-member-type参数用于控制NodePort类型的Service下发后的member类型。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/ed3fd3d88e8919b67a656d8d8ba6bc4e55cca67f)
* 调整prometheus监控指标，追加事件处理耗时监控。[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/bb4f08d38debcd9b266c55d5c62a059fdf360bb6)
* 增加周期性任务机制用于处理BIG-IP配置保存。 [commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/9a6887e50e6f7cb7354af05454c3ebf4710323c9)


## Release: **2.14.1-20230712**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.14.1-20230712](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 延迟pod删除时的arp删除(针对flannel vxlan)，确保业务在pod删除前的联通性 [commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/1e1584c637ca8b69d0000bd3a67a90ed27d0f12d)
* 实现架构调整:
  * 使用controller-runtime取代informer实现对资源事件的监听。commits [1](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/ee4587dbd607fa0f67708e953a609c4bb15a1360), [2](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/f278ad84f87a8d5a95a1afaafaba59b8768a37d7)
  * 串行化处理BIG-IP iControl Rest请求，保证BIG-IP下发一致性。 commits [1](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/ed9200014929a5c74df8dc65695f2d9569fde8fa), [2](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/193631d9d55e841b98ceb3a2716f98efc045b2c1), [3](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/eb3d06beb0733ba782cf84954e980f06fd54c202)
* 修复当资源事件量大时的卡死问题，[commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/75689944661eb462bbd0d14c6cbc1e877eea663c)
* 增加/hook/settings API实现部分配置项动态化：log-level sys-save-interval health-check-interval。commits [1](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/c6a8746806f1d3259c7e2cd618cba6c25ef5f1cf), [2](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/a5dbeed25093a89c7b176ec64b5bd220d5adf4fe)

* 性能优化，减少不必要的资源触发事件。  [commit](https://gitlab.f5net.com/cis-c/f5-kic/-/commit/7fab8d4ba648c704e42b201c55590114469665d8)


## Release: **2.13.1-20230629**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.13.1-20230629](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增加/hook/prestop API支持业务的优雅退出以确保100%SLA ([issue](https://github.com/f5se/cis-c-docs/issues/57))。
* 增加对Cilium CNI的支持 ([issue](https://github.com/f5se/cis-c-docs/issues/56))。


## Release: **2.12.2-20230621**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.12.2-20230621](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增加对flannel CNI中host-gw模式的支持 ([PR internal-only](https://gitee.com/zongzw/f5-kic/pulls/70)), ([issue](https://github.com/f5se/cis-c-docs/issues/53))。

## Release: **2.12.1-20230608**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.12.1-20230608](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 完善NodePort支持，ServiceType可以在NodePort和ClusterIP两种类型间自由切换部署。([issue](https://github.com/f5se/cis-c-docs/issues/52) and [PR](https://gitee.com/zongzw/f5-kic/pulls/69))。
* 增加cis.f5.com/as3-tenant as3-app as3-pool 三个label的下发响应（被移除、新加时都会触发资源下发调整）([issue](https://github.com/f5se/cis-c-docs/issues/51))。
* 优化以下资源的下发逻辑(rest请求+事务)：net/arp net/ndp ltm/node

## Release: **2.11.1-20230216**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.11.1-20230216](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 修复服务标签绑定异常([issue](https://github.com/f5se/cis-c-docs/issues/48) and [PR](https://gitee.com/zongzw/f5-kic/pulls/67))。
* 完善具备request id的日志输出([PR](https://gitee.com/zongzw/f5-kic/pulls/66))。

## Release: **2.10.1-20221201**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.10.1-20221201](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增强底层模块管理，业务下发和AS3解析逻辑独立为[f5-bigip-rest](https://gitee.com/zongzw/f5-bigip-rest)和[f5-as3-parsing](https://gitee.com/zongzw/f5-as3-parsing)
* 增强安全交付能力，配置非root用户运行CIS-C
* 优化iControlRest生成逻辑
* 增强对函数耗时监控能力

## Release: **2.10.1-20221031**

### Docker Image

[f5devcentral/k8s-bigip-ctlr-c:2.10.1-20221031](https://hub.docker.com/r/f5devcentral/k8s-bigip-ctlr-c)

[f5devcentral/cis-c-as3-parser:latest](https://hub.docker.com/r/f5devcentral/cis-c-as3-parser)

### Release Notes

* 增强资源下发的租户管理
* 增强IPv6支持，并完善相关文档
* 其他功能修复和性能检测调优

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
