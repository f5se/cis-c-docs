# 下载与安装

安装F5 CIS-C。 参考下述yaml文件。按需求调整image版本等参数信息（详情参考[启动参数列表](/Architecture/parameters/)）。kubectl create -f my-deployment.yaml。
```
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
        - name: k8s-bigip-ctlr-c-pod
          image: "f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220830"
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
            # if flannel-name is set, meanning trying to use flannel mode
            # and customer needs to create related tunnel on BIGIP first.
            # "--flannel-name=fl-tunnel",
            # "--namespace=default",
            # "--namespace=namespace-1",
            # "--namespace-label=resource.zone=deployment",
            "--hub-mode",
            "--ignore-service-port"
          ]

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
    - name: k8s-bigip-ctlr-c-debug-dumps
      protocol: TCP
      port: 8082
      targetPort: 8082
      nodePort: 30082
  type: NodePort
```
