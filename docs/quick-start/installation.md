# 下载与安装

通过阅读本部分，我们可以了解到：

* 部署CIS-C应用实体Deployment的方法。
* 部署CIS-C应用的服务Service，用于外部与CIS-C交互。

调整以下yaml文件中CIS-C启动参数部分，并执行:

```shell
$ kubectl create -f deploy-cis-c.yaml
```

deploy-cis-c.yaml内容为：

```yaml
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-bigip-ctlr-c
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: k8s-bigip-ctlr-c-pod
  template:
    metadata:
      name: k8s-bigip-ctlr-c-pod
      labels:
        app: k8s-bigip-ctlr-c-pod
    spec:
      serviceAccountName: k8s-bigip-ctlr
      containers:
        # kubectl logs -f deployment/k8s-bigip-ctlr-c -c k8s-bigip-ctlr-c-pod -n kube-system
        - name: k8s-bigip-ctlr-c-pod
          image: f5devcentral/k8s-bigip-ctlr-c:2.14.6-20230728
          imagePullPolicy: IfNotPresent
          env:
            - name: BIGIP_USERNAME
              valueFrom:
                secretKeyRef:
                  name: bigip-login
                  key: username
            - name: BIGIP_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bigip-login
                  key: password
            - name: BIGIP_URL
              valueFrom:
                secretKeyRef:
                  name: bigip-login
                  key: url
          command: ["/f5-kic-linux"]
          args: [
            "--bigip-username=$(BIGIP_USERNAME)",
            "--bigip-password=$(BIGIP_PASSWORD)",
            "--bigip-url=$(BIGIP_URL)",
            "--log-level=debug",
            # "--flannel-name=fl-tunnel", # for flannel vxlan mode
            "--hub-mode=true",  # ok as well: "--hub-mode"
            "--ignore-service-port"
          ]

        # validate with: curl http://localhost:8081/validate
        - name: as3-parser
          image: "f5devcentral/cis-c-as3-parser:latest"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8081
              protocol: TCP

--- 

# the following service is optional.
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
  type: NodePort
```

注意，
* 按需求调整image版本等参数信息（详情参考[启动参数列表](/Architecture/parameters/)）.
* 如果hub.docker.com网络不可达，需要提前下载docker images，或者建立私有docker repository。
* CIS-C启动后可以通过 `kubectl logs -f deployment/k8s-bigip-ctlr-c -c k8s-bigip-ctlr-c-pod -n kube-system`查看运行状态。