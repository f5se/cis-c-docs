# 准备工作

### 1. BIG-IP 设备 (以下vxlan相关步骤仅针对flannel模式。如果为calico等模式，则不需要该步骤)

  - 创建VXLAN

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

  - 创建本地VLAN及SelfIP

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

    注意：*使用BIG-IP实际traffic 接口IP替换 10.250.16.127*

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

  - 创建rolebinding

    `kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr`

    或者使用以下yaml创建：

    ```
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: k8s-bigip-ctlr-clusteradmin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
      - kind: ServiceAccount
        name: k8s-bigip-ctlr
        namespace: kube-system
    ```

    将以上内容放入role-binding.yaml文件中, 使用 `kubectl apply -f role-binding.yaml`命令执行文件内容。

  - 创建BIG-IP虚拟节点(仅在flannel模式下需要本步骤)

    创建bigip1 虚拟节点，打通BIG-IP节点到k8s的vxlan网络。

    使用以下yaml配置文件bigip_node.yaml：

    ```
    apiVersion: v1
    kind: Node
    metadata:
      name: bigip1
      annotations:
        #Replace IP with Self-IP for your deployment
        flannel.alpha.coreos.com/public-ip: "10.250.18.105"
        #Replace MAC with your BIGIP Flannel VXLAN Tunnel MAC
        flannel.alpha.coreos.com/backend-data: '{"VtepMAC":"fa:16:3e:d5:28:07"}'
        flannel.alpha.coreos.com/backend-type: "vxlan"
        flannel.alpha.coreos.com/kube-subnet-manager: "true"
    spec:
      #Replace Subnet with your BIGIP Flannel Subnet
      podCIDR: "10.42.20.0/24"
    ```

    其中 fa:16:3e:d5:28:07 可以在BIG-IP上使用此命令获取：

    `show net tunnels tunnel fl-tunnel all-properties`

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