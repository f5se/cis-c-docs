# 版本发布

## Release: **2.9.1-20220804**

### Image
    f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220804

### Release Note

    部署与环境支持
    * 支持inCluster（更通用）和--kubeconfig两种运行模式
    * 支持k8s集群业务配置方式：deployment+service，应用交付配置方式：configmap
    * 支持k8s网络CNI类型：flannel 和calico，可通过参数--flannel-name设定。

    功能与行为支持
    * 支持多namespace下业务配置监控。
    * 支持使用namespace-label参数设定监控范围（多与hub-mode 参数联合使用）。
    * 支持hub-mode方式，以便业务和应用交付配置可以在不同的namespace中分别进行。
    * 支持服务绑定保护，避免误操作带来的服务覆盖异常。
    * 支持默认服务端口绑定：当configmap应用配置中无端口指定时，自动选择服务端口。
    * 支持增量配置方式，以提高下发速度。
    * 支持BIG-IP上扩展应用交付能力包括（及子类型配置）：
    * virtual
    * virtual-address
    * snatpool
    * pool/member/node
    * monitor：gateway_icmp tcp udp http
    * profile: tcp ftp udp http clientssl oneconnect
    * persist: cookie source_addr
    * irule
    * 支持NAT64 即virtual(ipv6)到pool member(ipv4)转换配置方式。

    运维与升级支持
    * 支持从CIS升级到CIS-C，需要借助kic-tool工具完成升级迁移。
    * 支持日志级别输出、和任务跟踪（request_id）。
    * 支持prometheus下发性能数据监控。
