# 产品架构介绍

F5 CIS-C程序监控K8s中Configmap（包含AS3模板格式的交付配置）和Endpoints资源（包含客户应用服务信息），将其转换并下发为BIG-IP上的应用交付能力。之后，BIG-IP作为客户的应用服务入口，提供L4-L7的业务及流量的管控。

工作模型如下图所示：

![image](working-model.png)

所采用的技术栈包括：

![image](tech-stack.png)