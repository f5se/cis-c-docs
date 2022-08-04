# 版本发布

## Release: **2.9.1-20220804**

### Image
    f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220804

### Release Notes

#### 部署与环境支持
* 支持集群内和集群外运行两种模式
* 支持k8s集群业务配置方式：deployment+service，应用交付（应用对外发布）配置方式：configmap([F5 AS3](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/)作为数据主体)
* 支持k8s CNI类型：Flannel vxlan或其它underlay模式CNI（如Calico路由模式，Kube-ovn underlay模式等），配置细节可参考[这里](/Architecture/parameters/)
* 支持广泛的k8s版本，如您发现k8s的版本支持问题，请[联系我们](/Support-and-contact/)
* 暂不支持Openshift

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
* 兼容用户当前在CIS中已经发布的configmap yaml格式，用户在CIS-C中无法修改应用发布yaml
* 支持从CIS升级到CIS-C，需要借助kic-tool工具完成升级迁移
* 支持日志级别输出、和任务跟踪（request_id）
* 支持与Prometheus集成，采集控制器性能数据
* 在有效管理下，F5 CIS-C可以与F5 CIS同时运行，实现更灵活丰富的场景应对
