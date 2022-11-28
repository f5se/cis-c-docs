# 准备工作

### 1. BIG-IP 设备 

> 以下vxlan相关步骤仅针对flannel模式。如果为calico等underlay模式，则不需要该步骤

  - 创建VXLAN profile

    ```
    create net tunnels vxlan /Common/fl-vxlan {
        app-service none
        flooding-type none
        port 8472
    }
    ```

  - 创建VXLAN Tunnel 及VTEP IP

    ```
    create net tunnels tunnel /Common/fl-tunnel {
        key 1
        local-address 10.250.16.127
        profile /Common/fl-vxlan
    }
    ```

    注意：*使用BIG-IP实际traffic 接口IP替换 10.250.16.127*

  - (可选) 创建VXLAN V6 Tunnel 及VTEP IP. 仅当使用 IPV6 tunnel 场景时才需要。

    ```
    create net tunnels tunnel /Common/fl-tunnel6 {
        key 1
        local-address 2021:15:0:0:0:0:0:125
        profile /Common/fl-vxlan
    }
    ```

    注意：*使用BIG-IP实际的IP替换 2021:15:0:0:0:0:0:125*

  - 创建VXLAN Tunnel SelfIP

    ```
    create net self /Common/fl-vxlan-selfip {
        address 10.42.20.1/16
        allow-service all
        traffic-group /Common/traffic-group-local-only
        vlan /Common/fl-tunnel
    }
    ```

    注意这里的10.42.20.1/16的前两位通过指令`kubectl get node -o yaml | grep podCIDR`获取，第3位任选数值不与已有node重复即可。

  - (可选) 创建VXLAN Tunnel IPV6 SelfIP . 仅当使用 IPV6 tunnel 场景时才需要。

    ```
    create net self /Common/fl-vxlan-selfip-6 {
        address 2021:118:2:2::/60
        allow-service all
        traffic-group /Common/traffic-group-local-only
        vlan /Common/fl-tunnel6
    }
    ```

  - 创建本地VLAN及SelfIP. 其中 vlan-traffic-6 这个IPV6 地址可选，仅当使用 IPV6 tunnel 场景时才需要。

    ```
    create net vlan /Common/vlan-traffic {
        interfaces {
            1.1 { }
        }
        sflow {
            poll-interval-global no
            sampling-rate-global no
        }
        tag 4094
    }
    ```

    ```
    create net self /Common/vlan-traffic {
        address 10.250.16.127/24
        traffic-group /Common/traffic-group-local-only
        vlan /Common/vlan-traffic
    }
    ```

    ```
    create net self /Common/vlan-traffic-6 {
        address 2021:15::125/32
        traffic-group /Common/traffic-group-local-only
        vlan /Common/vlan-traffic
    }
    ```

    注意：*使用BIG-IP实际 traffic 接口IP分别替换 V4 和 V6 地址。*

### 2. Kubernetes

  - 创建serviceaccount

    `kubectl create serviceaccount k8s-bigip-ctlr -n kube-system`

    或者使用以下yaml创建：

    ```
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: k8s-bigip-ctlr
      namespace: kube-system
    ```

    将以上内容放入service-account.yaml文件中, 使用 `kubectl apply -f service-account.yaml`命令执行文件内容。

  - 创建BIG-IP登陆密码 (视需要修改用户名,密码以及URL等)

    ```
    kubectl create secret generic bigip-login
      -n kube-system
      --from-literal=username=admin
      --from-literal=password=mypassword
      --from-literal=url=https://10.10.10.10:443
    ```

    注意：*使用BIG-IP实际url及密码替换示例中的https://10.10.10.10:443及mypassword*

    或者使用以下yaml创建：
    ```
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: bigip-login
      namespace: kube-system
    data:
      url: aHR0cHM6Ly8xMC4xMC4xMC4xMDo0NDM=   # base64 encoded 'https://10.10.10.10:443'
      username: YWRtaW4=                      # base64 encoded 'admin'
      password: bXlwYXNzd29yZA==              # base64 encoded 'mypassword'
    type: Opaque
    ```
    base64 编码指令为 `echo -n xxxxx | base64`

    将以上内容放入secret.yaml文件中, 使用 `kubectl apply -f secret.yaml`命令执行文件内容。

  - 创建 Cluster Role 和 Cluster Role Binding

    参考以下yaml例子创建。可根据实际需要修改rbac权限范围。

    ```
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

  - 创建BIG-IP虚拟节点(仅在flannel模式下需要本步骤)

    创建bigip1 虚拟节点，打通BIG-IP节点到k8s的vxlan网络。

    使用以下yaml配置文件bigip1.yaml：

    ```
    apiVersion: v1
    kind: Node
    metadata:
      name: bigip1
      annotations:
        # Replace IP with Self-IP for your deployment
        flannel.alpha.coreos.com/public-ip: "10.250.18.105"
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

    其中 mac 地址可以在BIG-IP上使用tmsh命令获取：

    `show net tunnels tunnel fl-tunnel all-properties`

    `show net tunnels tunnel fl-tunnel6 all-properties`

    将以上内容放入bigip1.yaml文件中, 使用 `kubectl apply -f bigip1.yaml`命令执行文件内容。

### 3. Prometheus 监控（可选）

  F5 CIS-C 程序开启了8080端口用于Prometheus收集metrics

  ```
    # expose the Prometheus port with NodePort
    apiVersion: v1
    kind: Service
    metadata:
      name: k8s-bigip-ctlr-c-svc
      namespace: kube-system
    spec:
      selector:
        app: k8s-bigip-ctlr-c-pod
      ports:
        - name: k8s-bigip-ctlr-c-metrics
          protocol: TCP
          port: 8080
          targetPort: 8080
          nodePort: 30080
  ```

  所以，在Prometheus服务器端，需要在配置中添加配置项：

  ```
    - job_name: "f5-cis-c"
      static_configs:
        - targets: ["10.250.18.102:30080"]
  ```
  其中 10.250.18.102 为F5 CIS-C程序所在Node的IP。

### 4. calico 配置参考（可选）仅当calico模式时需要。

  注意： 检查相应的self IP 地址中，Port Lockdown 设置为 “Allow All” 或者添加上TCP custom port 端口179。

  - BIGIP上操作：进入Network -> Route domain. 进入 Route Domain 0配置页面中，选中BGP将其移入到enabled列表中。 然后ssh 登录到BIGIP上，执行：

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

  - K8S master节点上操作：
    ```
    curl -O -L https://github.com/projectcalico/calicoctl/releases/download/v3.10.0/calicoctl
    chmod +x calicoctl
    sudo mv calicoctl /usr/local/bin
    sudo mkdir /etc/calico
    vim /etc/calico/calicoctl.cfg
    ```

    calicoctl.cfg 文件内容参考如下。修改成环境里正确的kubeconfig 的路径。
    ```
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
    ```
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
