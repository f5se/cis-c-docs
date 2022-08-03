# 下载与安装

安装F5 CIS-C。 参考下述yaml文件。按需求修改image版本、等参数信息（详情参考启动参数列表）。kubectl create -f my-deployment.yaml。
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
      nodeSelector:
        node-role.kubernetes.io/controlplane: "true"
        # node-role.kubernetes.io/master: "true"
      containers:
        # kubectl logs -f deployment/k8s-bigip-ctlr-c -c k8s-bigip-ctlr-c-pod -n kube-system
        - name: k8s-bigip-ctlr-c-pod
          image: "zongzw/k8s-bigip-ctlr-c:2.9.01-20220726"
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
            "--flannel-name=fl-tunnel",
            # "--namespace=default",
            # "--namespace=namespace-1",
            # "--namespace-label=resource.zone=deployment",
            "--hub-mode",
            "--ignore-service-port"
          ]

        # validate with: curl http://localhost:8081/validate
        - name: as3-parser
          image: "zongzw/as3-parser:latest"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8081
              protocol: TCP

        # kubectl exec -it deployment/k8s-bigip-ctlr-c -c debug-pod -n kube-system -- bash
        - name: debug-pod
          image: "zongzw/k8s-bigip-ctlr-c-debug-host:latest"
          imagePullPolicy: IfNotPresent
          command:
            - /bin/bash
            - -c
          args:
            - while true; do sleep 1; done
--- 

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
