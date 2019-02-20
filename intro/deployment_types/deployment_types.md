# 部署类型

Envoy 有多种使用场景，其中更多情况下会作为 *mesh* 被部署在基础设施的不同主机上。本章节将按照复杂性上升的顺序讨论 envoy 的三种推荐部署类型。

- 仅服务之间
  - [服务间 egress listener](service_to_service.md#service-to-service-egress-listener)
  - [服务间 ingress listener](service_to_service.md#service-to-service-ingress-listener)
  - [可选外部服务 egress listener](service_to_service.md#optional-external-service-egress-listeners)
  - [发现服务集成](service_to_service.md#discovery-service-integration)
  - [配置模板](service_to_service.md#configuration-template)
- 服务间外加前端代理
  - [配置模板](front_proxy.md#configuration-template)
- 服务间、前端代理、双向代理
  - [配置模板](double_proxy.md#configuration-template)
