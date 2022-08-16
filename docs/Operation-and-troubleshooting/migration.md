# 迁移

所谓“迁移”为将K8S集群资源下发控制器从[CIS](https://clouddocs.f5.com/containers/latest/)切换为CIS-C的过程。

切换的目的是使用CIS-C取代CIS，接管K8S集群资源到BIG-IP的下发能力，即，切换完成后，已下发业务的变更及新的业务下发均由CIS-C接管，CIS退出。

## CIS-C迁移步骤

1. 停止CIS程序，确保BIG-IP上现有业务不再有新的变更。
2. 备份BIG-IP现有业务配置，实现方式可以通过ucs或者其他备份方式，以备切换失败时恢复。
3. 运行cis-c-tool程序，具体使用方法参见[工具使用方法](https://gitee.com/zongzw/kic-tool#%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95)，特别注意的是需要添加参数`--output bigip`，必要时加参数`--overwrite`，详见“工具使用方法”。

   此步执行的作用是将BIG-IP上现有的已经下发的资源配置，收集起来，以特定格式（JSON）存入BIG-IP internal类型的Data Group中。

   执行完本步骤后，在 BIG-IP "Local Traffic" -> "iRules" -> "Data Group List"中，可以看到以“f5-kic_”开头的partition信息，例如“f5-kic_namespaceA”。
   
   打开此条目后可以看到“String Records”，“String Records” 中包含多个以“rest.x”或者“as3.x”命名的条目，他们的值均以base64编码。x为数字编号，这是因为单个“String Record”有65536字节长度限制，所以完整的信息被切分成1024字节长度，并以“.x”编号。

   rest.x 表示已经解析完成的CIS-C可以识别的信息。

   as3.x 表示configmap中原始的as3配置。
   
   运行cis-c-tool程序后，Data Group中并不会出现as3.x类型的条目，因为此时cis-c-tools仅能获取CIS下发后的BIG-IP上资源状态，还无法拿到as3原始配置信息，只能拿到rest.x 表示的已下发状态。当启动CIS-C后，as3.x部分由CIS-C补齐。

   将rest.x 内容拼装还原，我们可以看到cis-c-tool程序往BIG-IP上保存了什么内容，以何种格式。 举例来说，参见以下注释部分阐述。

   ```
   {
      # partition folder下的资源，并非所有资源都在subPath下
      "": {
         # virtual-address 类型资源
         # “ltm/virtual-address”为iControlRest类型
         # 192.0.2.11为资源名称
         # 后边{...}为资源的属性
         "ltm/virtual-address/192.0.2.11": {
               "address": "192.0.2.11",
               "arp": "enabled",
               ...
               "unit": 1
         }
      },
      # cafe为folder名称，folder本身也是一种资源
      # folder内为各个资源的iControlRest类型及资源名称、属性
      "cafe": {
         "ltm/pool/coffee_pool": {
               "allowNat": "yes",
               ...
               "subPath": "cafe"
         },
         "ltm/profile/client-ssl/cafetls": {
               "alertTimeout": "indefinite",
               ...
               "uncleanShutdown": "enabled"
         },
         "ltm/virtual/https_vs": {
               "addressStatus": "yes",
               ...
               "vsIndex": 12238
         },
         "sys/file/ssl-cert/coffeecert.crt": {
               ...
               "version": 3
         },
         "sys/file/ssl-cert/teacert.crt": {
               ...
               "version": 3
         },
         "sys/file/ssl-key/coffeecert.key": {
               ...
               "updatedBy": "admin"
         },
         "sys/file/ssl-key/teacert.key": {
               ...
               "updatedBy": "admin"
         }
      }
      # 一个partition folder下可以有多个folder，取决于as3声明方式。
   }
   ```

   将as3解析为符合以上格式的结构体是CIS-C实现下发的关键、基础。
   
   **运行cis-c-tool程序的根本目的，就是将BIG-IP上现有的资源信息组织起来（JSON）以internal Data Group的形式告知CIS-C程序。**
   
   **只有这样，CIS-C才能继续对资源进行Create Update Delete操作。**

4. 启动CIS-C程序，完成迁移。维护人员可自行观测日志，并观察BIG-IP上已下发资源情况。

   CIS-C启动后会首先加载interval Data Group中以`f5-kic_`开头的数据，将其作为已下发资源状态。有了这份信息，CIS-C可以：

   * 避免CIS-C重启后所有资源重新下发，这是因为interval Data Group信息可以作为上次下发的fingerprint，CIS-C每次下发前均会对比是否有变化。

   * 在其他下发程序下发基础上继续增量更新。虽然CIS-C并不能知道之前下发程序（如CIS）是如何下发的，但通过cis-c-tool可以知道下发程序操作后BIG-IP上的资源状态。在此状态上，CIS-C便可以知道各种资源的关联关系（例如virtual-pool-member关联关系），就可以继续CRUD了。