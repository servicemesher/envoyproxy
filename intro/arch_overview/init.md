# 初始化

Envoy 在启动时的初始化是很复杂的。本章将在高级别解释这个过程是如何工作的。以下会在任何监听器启动监听并接收新连接前发生。

- 启动期间，[集群管理器](cluster_manager.md#arch-overview-cluster-manager) 会首先进行多阶段初始化，首先初始化静态/ DNS 集群，然后是预定义的 [SDS](dynamic_configuration.md#arch-overview-dynamic-config-sds) 集群。 然后，如果适用，它会初始化 [CDS](dynamic_configuration.md#arch-overview-dynamic-config-cds) ， 等待响应（或失败）， 并为 CDS 提供的集群执行相同主/次初始化。
- 如果集群使用 [主动健康检查](health_checking.md#arch-overview-health-checking) ，Envoy也会执行单个主动HC轮次。
- 集群管理器初始化完成后， [RDS](dynamic_configuration.md#arch-overview-dynamic-config-rds) 和 [LDS](dynamic_configuration.md#arch-overview-dynamic-config-lds) 将初始化（如果适用）。在LDS / RDS请求至少有一次响应（或失败）之后，服务器才开始接受连接。
- 如果 LDS 本身返回需要 RDS 响应的监听器，则Envoy会进一步等待，直到收到 RDS 响应（或失败）。请注意，这个过程发生在未来的每个通过 LDS 添加的监听器上，并且被称为 [监听器热身](../../configuration/listeners/lds.md#config-listeners-lds)。
- 在先前所有的步骤发生之后，监听器开始接受新的连接。该流程可确保在热启动期间新流程完全能够在旧流程排除之前接受并处理新连接。