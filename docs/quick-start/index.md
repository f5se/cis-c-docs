# 准备工作

CIS-C以K8S Deployment的形式部署在`kube-system`命名空间中，通过[List-Watch](https://github.com/topics/list-watch)机制监听k8s侧发生的资源事件，并将其转化并下发为BIG-IP应用交付能力。

为了部署并正常运行CIS-C，我们需要：

* K8S侧：创建ServiceAccount
* K8S侧：分配相应ClusterRole权限
* K8S侧：创建BIG-IP登陆信息
* BIG-IP侧：根据CNI类型配置基础网络

*注意：CIS-C与CIS采用相同的权限配置方式，更多细节可详见[installing CIS Manually](https://clouddocs.f5.com/containers/latest/userguide/cis-installation.html#installing-cis-manually)。*

## K8S侧创建相关资源

  - 创建serviceaccount

    ```shell
    $ kubectl create serviceaccount k8s-bigip-ctlr -n kube-system
    ```

    或者使用以下yaml创建：

    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: k8s-bigip-ctlr
      namespace: kube-system
    ```

    将以上内容放入service-account.yaml文件中, 使用 `kubectl apply -f service-account.yaml`命令执行文件内容。

  - 创建 Cluster Role 和 Cluster Role Binding

    参考以下yaml例子创建。可根据实际需要修改rbac权限范围。

    ```yaml
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: bigip-ctlr-clusterrole
    rules:
    - apiGroups: ["", "extensions", "networking.k8s.io"]
      resources: ["nodes", "services", "endpoints", "namespaces", "ingresses", "pods", "ingressclasses"]
      verbs: ["get", "list", "watch"]
    - apiGroups: ["", "extensions", "networking.k8s.io"]
      resources: ["configmaps", "events", "ingresses/status", "services/status"]
      verbs: ["get", "list", "watch", "update", "create", "patch"]
    - apiGroups: ["", "extensions"]
      resources: ["secrets"]
      verbs: ["get", "list", "watch"]

    ---

    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: bigip-ctlr-clusterrole-binding
      namespace: kube-system
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: bigip-ctlr-clusterrole
    subjects:
    - apiGroup: ""
      kind: ServiceAccount
      name: k8s-bigip-ctlr
      namespace: kube-system
    ```

    将以上内容放入rbac.yaml文件中, 使用 `kubectl apply -f rbac.yaml`命令执行文件内容。

  - 创建BIG-IP登陆信息

    ```shell
    $ kubectl create secret generic bigip-login
      -n kube-system
      --from-literal=username=admin
      --from-literal=password=mypassword
      --from-literal=url=https://10.10.10.10:443
    ```

    注意：*使用BIG-IP实际url及密码替换示例中的https://10.10.10.10:443及mypassword*

    或者,使用以下yaml创建：
    ```yaml
    ---

    apiVersion: v1
    kind: Secret
    metadata:
      name: bigip-login
      namespace: kube-system
    stringData:
      url: https://10.10.10.10:443
      username: admin
      password: mypassword
    type: Opaque
    ```

    将以上内容放入secret.yaml文件中, 使用 `kubectl apply -f secret.yaml`命令执行文件内容。

## BIG-IP侧配置基础网络

目前CIS-C支持3中网络CNI：

* Flannel网络：vxlan模式和host-gw模式
* Calico网络：BGP模式
* Cilium网络：vxlan模式

### Flannel网络

#### VXLAN模式

> 当K8S网络为Flannel VXLAN时，CIS-C的启动参数中需要使用`--flannel-name`或者`--flannel-name-v6`指定本部分创建的tunnel：`fl-tunnel`。

  - 创建VXLAN profile

    ```shell
    create net tunnels vxlan /Common/fl-vxlan {
        app-service none
        flooding-type none
        port 8472
    }
    ```

  - 创建VXLAN Tunnel 及VTEP IP

    ```shell
    create net tunnels tunnel /Common/fl-tunnel {
        key 1
        local-address 10.250.16.127
        profile /Common/fl-vxlan
    }
    ```

    注意：*使用BIG-IP实际traffic 接口IP替换 10.250.16.127*

  - (可选) 创建VXLAN V6 Tunnel 及VTEP IP。
  
    *仅当使用 IPV6 tunnel 场景时才需要。*

    ```shell
    create net tunnels tunnel /Common/fl-tunnel6 {
        key 1
        local-address 2021:15:0:0:0:0:0:125
        profile /Common/fl-vxlan
    }
    ```

    注意：*使用BIG-IP实际的IP替换 2021:15:0:0:0:0:0:125*

  - 创建VXLAN Tunnel SelfIP

    ```shell
    create net self /Common/fl-vxlan-selfip {
        address 10.42.20.1/16
        allow-service all
        traffic-group /Common/traffic-group-local-only
        vlan /Common/fl-tunnel
    }
    ```

    注意这里的10.42.20.1/16的前两位通过指令`kubectl get node -o yaml | grep podCIDR`获取，第3位任选数值不与已有node重复即可。

  - (可选) 创建VXLAN Tunnel IPV6 SelfIP。
  
    *仅当使用 IPV6 tunnel 场景时才需要。*

    ```shell
    create net self /Common/fl-vxlan-selfip-6 {
        address 2021:118:2:2::/60
        allow-service all
        traffic-group /Common/traffic-group-local-only
        vlan /Common/fl-tunnel6
    }
    ```

  - 创建本地VLAN及SelfIP. 其中 vlan-traffic-6 这个IPV6 地址可选，仅当使用 IPV6 tunnel 场景时才需要。

    ```shell
    create net vlan /Common/vlan-traffic {
        interfaces {
        # interfaces add {  # 不同TMOS版本的命令参数稍有不同
            1.1 { }
        }
        sflow {
            poll-interval-global no
            sampling-rate-global no
        }
        tag 4094
    }
    ```

    ```shell
    create net self /Common/vlan-traffic {
        address 10.250.16.127/24
        traffic-group /Common/traffic-group-local-only
        vlan /Common/vlan-traffic
    }
    ```

    ```shell
    create net self /Common/vlan-traffic-6 {
        address 2021:15::125/32
        traffic-group /Common/traffic-group-local-only
        vlan /Common/vlan-traffic
    }
    ```

    注意：*使用BIG-IP实际 traffic 接口IP分别替换 V4 和 V6 地址。*

  - K8S侧创建BIG-IP虚拟节点资源

    创建bigip1 虚拟节点，打通BIG-IP节点到k8s的vxlan网络。

    使用以下yaml配置文件bigip1.yaml：

    ```yaml
    apiVersion: v1
    kind: Node
    metadata:
      name: bigip1
      annotations:
        # Replace IP with Self-IP for your deployment
        flannel.alpha.coreos.com/public-ip: "10.250.16.127"
        # uncomment the following line if using v6 tunnel and modify bigip v6 address
        # flannel.alpha.coreos.com/public-ipv6: "2021:15::125"
        # Replace MAC with your BIGIP Flannel VXLAN Tunnel MAC
        flannel.alpha.coreos.com/backend-data: '{"VtepMAC":"fa:16:3e:d5:28:07"}'
        # uncomment the following line if using v6 tunnel and modify mac accordingly
        # flannel.alpha.coreos.com/backend-v6-data: '{"VtepMAC":"fa:16:3e:d5:28:07"}'
        flannel.alpha.coreos.com/backend-type: "vxlan"
        flannel.alpha.coreos.com/kube-subnet-manager: "true"
    spec:
      # Replace Subnet with your BIGIP Flannel Subnet
      podCIDR: "10.42.20.0/24"
      # uncomment the following 3 lines if using v6 tunnel and modify CIDRs using real data
      #podCIDRs:
      #- "10.42.20.0/24"
      #- "2021:118:2:2::/64"
    ```

    其中 mac 地址`fa:16:3e:d5:28:07`可以在BIG-IP上使用tmsh命令获取：

    ```shell
    $ show net tunnels tunnel fl-tunnel all-properties
    $ show net tunnels tunnel fl-tunnel6 all-properties # 如果是ipv6网络时
    ```

    将以上内容放入bigip1.yaml文件中, 使用 `kubectl apply -f bigip1.yaml`命令执行文件内容。

#### host-gw模式

flannel的vxlan模式属于隧道方式，也就是在udp的基础之上，构建虚拟网络。

host-gw模式不同，host-gw模式属于路由的方式，由于没有经过任何封装，纯路由实现，数据只经过协议栈一次，因此性能比较高。

host-gw模式下并不需要预先在K8S侧创建BIG-IP虚拟节点，也不需要在BIG-IP创建相应tunnel。

host-gw模式下，数据包走纯路由模式，因此需要在BIG-IP建立数据返回K8S的路由信息。

WebUI上的配置项在Network -> Routes下。

创建相关route的行为已经封装在CIS-C中，CIS-C会根据K8S nodes的情况动态创建相关routes。

### Calico网络

注意： 检查相应的self IP 地址中，Port Lockdown 设置为 “Allow All” 或者添加上TCP custom port 端口179。

  - BIG-IP侧开启BGP
  
    方式1：
    
    在BIG-IP WebUI上，导航至Network -> Route domain. 进入 Route Domain 0配置页面中，选中BGP将其移入到enabled列表中。 然后ssh 登录到BIGIP上，执行：

    ```
    imish
    enable
    config terminal
    router bgp 64512
    neighbor calico-k8s peer-group
    neighbor calico-k8s remote-as 64512
    neighbor <OTHER BIG-IP IP ADDRESS> peer-group calico-k8s
    neighbor <each k8s NODE IP ADDRESS> peer-group calico-k8s
    write
    end
    ```

    方式2：

    在BIG-IP TMSH中，执行：

    ```shell
    $ tmsh modify sys db tmrouted.tmos.routing value enable
    ```

    注意，两种方式互斥，只能选择其中一种开启BIG-IP的BGP路由功能。

  - K8S侧建立BIG-IP的BGP Peer节点：

    ```shell
    $ curl -O -L https://github.com/projectcalico/calicoctl/releases/download/v3.10.0/calicoctl
    chmod +x calicoctl
    sudo mv calicoctl /usr/local/bin
    sudo mkdir /etc/calico
    vim /etc/calico/calicoctl.cfg
    ```

    calicoctl.cfg 文件内容参考如下。修改成环境里正确的kubeconfig 的路径。
    ```yaml
    apiVersion: projectcalico.org/v3
    kind: CalicoAPIConfig
    metadata:
    spec:
      datastoreType: "kubernetes"
      kubeconfig: "/root/.kube/config"    ## 请修改为实际路径
    ```

    运行 calicoctl get nodes 以检查 calicoctl 安装完成。

    执行以下命令：
    ```
    cat << EOF | calicoctl create -f -
    apiVersion: projectcalico.org/v3
    kind: BGPConfiguration
    metadata:
      name: default
    spec:
      logSeverityScreen: Info
      nodeToNodeMeshEnabled: true
      asNumber: 64512
    EOF
    ```

    执行以下命令. 注意修改成真实的BIGIP地址。
    ```yaml
    cat << EOF | calicoctl create -f -
    apiVersion: projectcalico.org/v3
    kind: BGPPeer
    metadata:
        name: bgppeer-bigip1
    spec:
      peerIP: 192.2.3.4
      asNumber: 64512
    EOF
    ```

    配置完成后，在 BIG-IP imish命令行中，通过`show bgp neighbors` 查看neighbor状态，确认neighbor状态为`BGP state = Established`

    ```
    #show bgp neighbors
    BGP neighbor is 10.250.17.159, remote AS 64512, local AS 64512, internal link
    Member of peer-group calico-k8s for session parameters
      BGP version 4, remote router ID 10.250.17.159
      BGP state = Established, up for 00:00:43
      Last read 00:00:43, hold time is 90, keepalive interval is 30 seconds
      Neighbor capabilities:
        Route refresh: advertised and received (new)
        Address family IPv4 Unicast: advertised and received
      Received 5 messages, 0 notifications, 0 in queue
      Sent 4 messages, 0 notifications, 0 in queue
      Route refresh request: received 0, sent 0
      Minimum time between advertisement runs is 5 seconds
    For address family: IPv4 Unicast
      BGP table version 3, neighbor version 3
      Index 1, Offset 0, Mask 0x2
      calico-k8s peer-group member
      AF-dependant capabilities:
        Graceful restart: advertised, received

      Community attribute sent to this neighbor (both)
      1 accepted prefixes
      0 announced prefixes

    Connections established 1; dropped 0
    Graceful-restart Status:
      Remote restart-time is 120 sec

    Local host: 10.250.17.111, Local port: 55058
    Foreign host: 10.250.17.159, Foreign port: 179
    Nexthop: 10.250.17.111
    Nexthop global: fe80::f816:3eff:feee:83d2
    Nexthop local: ::
    BGP connection: non shared network

    BGP neighbor is 10.250.17.182, remote AS 64512, local AS 64512, internal link
    Member of peer-group calico-k8s for session parameters
      BGP version 4, remote router ID 10.250.17.182
      BGP state = Established, up for 00:00:29
      Last read 00:00:29, hold time is 90, keepalive interval is 30 seconds
      Neighbor capabilities:
        Route refresh: advertised and received (new)
        Address family IPv4 Unicast: advertised and received
      Received 5 messages, 0 notifications, 0 in queue
      Sent 4 messages, 0 notifications, 0 in queue
      Route refresh request: received 0, sent 0
      Minimum time between advertisement runs is 5 seconds
    For address family: IPv4 Unicast
      BGP table version 3, neighbor version 3
      Index 2, Offset 0, Mask 0x4
      calico-k8s peer-group member
      AF-dependant capabilities:
        Graceful restart: advertised, received

      Community attribute sent to this neighbor (both)
      1 accepted prefixes
      0 announced prefixes

    Connections established 1; dropped 0
    Graceful-restart Status:
      Remote restart-time is 120 sec

    Local host: 10.250.17.111, Local port: 42664
    Foreign host: 10.250.17.182, Foreign port: 179
    Nexthop: 10.250.17.111
    Nexthop global: fe80::f816:3eff:feee:83d2
    Nexthop local: ::
    BGP connection: non shared network
    ```

  亦或通过calicoctl `calicoctl node status`命令查看:

    ```
    # calicoctl node status
    Calico process is running.

    IPv4 BGP status
    +---------------+-------------------+-------+----------+-------------+
    | PEER ADDRESS  |     PEER TYPE     | STATE |  SINCE   |    INFO     |
    +---------------+-------------------+-------+----------+-------------+
    | 10.250.17.182 | node-to-node mesh | up    | 03:07:33 | Established |
    | 10.250.17.111 | global            | up    | 06:18:28 | Established |
    +---------------+-------------------+-------+----------+-------------+
    ```

  更多配置请参考：https://f5-k8s-istio-lab.readthedocs.io/en/latest/BGP/introduction.html


### Cilium网络

*更多详细配置可详见[BIG-IP Tunnel Setup for Cilium VTEP Integration](https://github.com/f5devcentral/f5-ci-docs/blob/master/docs/cilium/cilium-bigip-info.rst).*

Cilium 是一个用于透明保护部署在 Linux 容器管理平台（比如 Docker 和 Kubernetes）上的应用服务之间网络连接的开源软件。

它使用 eBPF 可以在 Linux 自身内部动态的插入一些控制逻辑，从而满足可观察性和安全性相关的需求。

Cilium vxlan模式下，我们需要在BIG-IP侧创建相同的vxlan tunnel。 与Flannel vxlan稍有不同的是：

* tunnel profile 的 flooding type 要设置为 `multipoint`。

  multipoint 使 BIG-IP 向K8S 发送 ARP 广播请求，以实现 Pod ARP 解析。

  ```shell
  $ tmsh create net tunnels vxlan fl-vxlan port 8472 flooding-type multipoint
  ```

* tunnel VNI key 要设置为 2。

  VNI 2 是 Cilium 的保留ID。

  ```shell
  $ tmsh create net tunnels tunnel flannel_vxlan key 2 profile fl-vxlan local-address 10.169.72.34
  ```

* tunnel 静态FDB 信息为0a:0a:0x:0x:0x:0x的形式，其中0x:0x:0x:0x部分为K8S各node节点16进制的IP地址。
  此部分配置在CIS-C启动时由CIS-C自动配置完成。

* 配置到pod CIDR的静态路由。

  配置信息为：

  ```yaml
  network: 10.0.0.0/16    # pod CIDR 总网段
  tmInterface: fl-tunnel  # 所有到10.0.0.0/16的流量均从tunnel fl-tunnel进出
  ```
  
  ```shell
  $ tmsh create net route 10.0.0.0 network 10.0.0.0/16 interface fl-tunnel
  ```

*更多cilium的使用和安装可参考[cilium官网](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/)或相关[博文](https://blog.csdn.net/zongzw/article/details/131244387).*
