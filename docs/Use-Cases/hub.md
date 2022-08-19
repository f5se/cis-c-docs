# 主页


<div style="display:none">

hub模式是一种很常见的应用场景，这种模式下，配置管理与服务管理可以相互独立，由不同运维人员在不同的namespace中处理。

。。。带来的好处。。和意义。。

plus..plus..plus..

参数含义

•	hubmode
用于指定 是否可以跨ns做cm与svc的关联
•	namespace namespace-label
用于圈定 控制器监听资源范围，准确说是
•	hubmode为true时表示namespace namespace-label指定的ns， configmap资源被监控，其他ns， svc ep被监控
•	hubmode 为false时表示所有资源的范围
如果namespace namespace-label参数未指定，则
•	hubmode 为true时 所有资源都被监控，资源跨ns可关联。
•	hubmode为false时，所有资源都被监控，资源跨ns不可关联。

</div>

欢迎使用CIS-C。

内容很快补充完毕
