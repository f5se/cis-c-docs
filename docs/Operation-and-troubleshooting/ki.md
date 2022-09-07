# 已知问题

* ` 问题1：直接删除ns导致程序崩溃重启，此问题极少偶然出现。`

   影响范围：资源依旧会被删除，但是程序重启会导致重启过程中的部分下发延时增加，增加下发失败的可能性。

   问题原因：直接删除namespace时，其下的各资源一并会被删除，程序在上传资源状态时因找不到namespace而失败。具体问题描述见：https://github.com/kubernetes/client-go/issues/1125

   解决方案：关闭资源状态上传功能，或寻求社区支持，协助定位并解决问题。


* ` 问题2：用户能够将AS3 declaration拆分为多个configmap分别下发。`

   影响范围：即，完整的AS3 declaration将由多个configmap 组成，每个configmap负责声明该AS3 declaration所指定的partition的一部分

   问题原因：据AS3 declaration 声明式的特性，同一declaration需要包含完整的配置信息。分拆declaration 违背了这一声明式语言的基本特性。

   解决方案：目前无法支持将AS3 declaration分拆下发的能力，这样非常容易引发数据不一致，增加程序维护复杂度。考虑放在将来特性中调研实现思路。


* ` 问题3：针对某租户（partition）的配置，必须通过某一确定configmap下发，configmap不可变更。`

   影响范围：partition x 通过 configmap a完成下发后，configmap b中即便包含了partition x的定义，下发后也不会生效。想要改动partition x，只能通过 configmap a。

   问题原因：此问题并非需要解决的问题，而是一种约束。configmap a下发后会形成对partition x的绑定关系，避免后续到来的configmap对既有业务产生错误影响。此约束对endpoints资源也有效，即，对某一个特定pool x的改动只能通过endpoints a。

   解决方案：如上所示，partition x 仅能通过configmap a下发，后续对partition x的修改、删除，均只能通过configmap a完成。
   
* ` 问题4：切换snat方式时报错，表现为从“snat”或者“self”切换为“auto”时下发失败`

  影响范围：从“snat”或者“self”切换为“auto”时下发失败，反之，即由“auto”变为“snat”或者“self”时不存在此问题。

  问题原因：此问题源自BIG-IP iControlRest的bug，将对应virtual的属性`sourceAddressTranslation`更新为'automap'后，对原有snatpool的关联关系没有及时清除，导致后续对snatpool的清理失败，日志报错：
  
  ```
  
    -1-> PATCH xxxx/mgmt/tm/ltm/virtual/myvirtual
    {... "sourceAddressTranslation":{"type":"automap"} ...}
    
    -2-> DELETE xxxx/mgmt/tm/ltm/snatpool/mysnatpool
    {"code":400,"message":"transaction failed:01070320:3: Snatpool mysnatpool is still referenced by a virtual server.","errorStack":[],"apiError":2}
    
  ```
  
   解决方案：

  方案一：登陆BIG-IP，手动修改对应virtual的`sourceAddressTranslation`属性为automap，然后执行kubectl apply 操作，完成从“snat”或“self”到“auto”的切换。
  	
  方案二：使用删除+创建的方式代替修改，即先执行kubectl delete -f 然后执行 kubectl create -f, 而不是直接kubectl apply -f做更新操作。

  > 关于业务连续性的考虑：因为从snat变为automap操作本身会造成业务断联，所以 delete->create 方式尚可接受（是否真的可接受，待评估）。


* `问题5：CIS-C不能感知其退出状态下的删除事件，即CIS-C启动之前的删除事件不会被监控`

  影响范围：在CIS-C启动前做过的k8s资源删除动作不会触发启动后CIS-C的删除行为。

  问题原因：k8s的list watch本质就是维持一个etcd的副本，而不是审计事务事件列表。CIS-C不能感知它不在的时候的删除事件，因为删除事件不能体现在k8s list watch列表中。

  解决方案：不要在CIS-C退出状态下做删除操作。
  
  如果已经进行过删除操作，只能通过手动删除BIG-IP上相关配置（不推荐），或者在启动CIS-C后重新下发一次配置(例如kubectl apply ..)后，再执行删除操作（例如kubectl delete ..），以触发CIS-C的删除行为（推荐）。