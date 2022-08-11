# 迁移

所谓“迁移”为将K8S集群资源下发控制器从[CIS](https://clouddocs.f5.com/containers/latest/)切换为CIS-C的过程。
切换完成后，已下发业务的变更及新的业务下发均由CIS-C接管，CIS退出。

## CIS-C迁移步骤

1. 停止CIS程序，确保BIG-IP上现有业务不再有新的变更。
2. 备份BIG-IP现有业务配置，实现方式可以通过ucs或者其他备份方式，以备切换失败时恢复。
3. 运行cis-c-tool程序，具体使用方法参见[工具使用方法](https://gitee.com/zongzw/kic-tool#%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95)，特别注意的是需要添加参数`--output bigip`
4. 启动CIS-C程序，记录日志，并观察BIG-IP上已下发资源情况。