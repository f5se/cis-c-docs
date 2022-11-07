# 下载与安装

### 1. 方式1（Installation Option 1）
参考下述yaml文件。按需求调整image版本等参数信息（详情参考[启动参数列表](/Architecture/parameters/)）.
Please update the related arguments in the yaml file and then install CIS-C by:
kubectl create -f my-deployment.yaml

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
          image: "f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220831"
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

### 2. 方式2（Installation Option 2）
在 args 参数列表中，建议应使用 --credentials-directory 参数来替代上述的 --bigip-username, --bigip-bigip-url
和 --bigip-password，以便更安全的保管BIGIP的用户名、密码等。

Another way to pass the BIGIP credentials. You can use "--credentials-directory" option
instead of passing --bigip-username, --bigip-bigip-url and --bigip-password in the args of the yaml file
above. When you use this argument, the controller looks for these three files in the specified directory: "username", "password" and "url". If any of these files do not exist, the controller falls back to using
the CLI arguments(--bigip-username, etc) as parameters. Each file should contain only the username, password,
and url, respectively. This is the more secure and thus prefered way because it can prevent the password
from being exposed via commands like 'ps -ef' in the k8s node.

  ```
  kubectl create secret generic bigip-credentials
    -n kube-system
    --from-literal=username=admin
    --from-literal=password=CHANGEME
    --from-literal=url=https://10.10.10.10:443
  ```

Then reference the secret above in the deployment yaml file. Please refer to "--credentials-directory"
, "volumeMounts" and "volumes" part below and update the other arguments as well.
Then run: kubectl create -f my-deployment.yaml

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
          image: "f5devcentral/k8s-bigip-ctlr-c:2.9.1-20220831"
          imagePullPolicy: IfNotPresent
          command: ["/f5-kic-linux"]
          args: [
            "--credentials-directory=/tmp/creds",
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
          volumeMounts:
            - name: bigip-creds
              mountPath: "/tmp/creds"
              readOnly: true

        - name: as3-parser
          image: "f5devcentral/cis-c-as3-parser:latest"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8081
              protocol: TCP

      volumes:
        - name: bigip-creds
          secret:
            secretName: bigip-credentials
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