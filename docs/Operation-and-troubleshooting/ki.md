# 已知问题

* ` 问题1：直接删除ns导致程序崩溃重启，此问题偶然出现。`

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
