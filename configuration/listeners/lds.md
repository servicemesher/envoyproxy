# 监听器发现服务（LDS）

监听器发现服务（LDS）是一个可选的 API，Envoy 将调用它来动态获取监听器。Envoy 将协调 API 响应，并根据需要添加、修改或删除已知的监听器。

监听器更新的语义如下：

- 每个监听器必须有一个独特的[名字](https://www.envoyproxy.io/docs/envoy/latest/api-v1/listeners/listeners.md#config-listeners-name)。如果没有提供名称，Envoy 将创建一个 UUID。要动态更新的监听器，管理服务必须提供监听器的唯一名称。

- 当一个监听器被添加，在参与连接处理之前，会先进入“预热”阶段。例如，如果监听器引用 [RDS](../http_conn_man/rds.md#config-http-conn-man-rds) 配置，那么在监听器迁移到 “active” 之前，将会解析并提取该配置。

- 监听器一旦创建，实际上就会保持不变。因此，更新监听器时，会创建一个全新的监听器（使用相同的侦听套接字）。新增加的监听者都会通过上面所描述的相同“预热”过程。

- 当更新或删除监听器时，旧的监听器将被置于 “draining（逐出）” 状态，就像整个服务重新启动时一样。监听器移除之后，该监听器所拥有的连接，经过一段时间优雅地关闭（如果可能的话）剩余的连接。逐出时间通过 [`--drain-time-s`](../../operations/cli.md#cmdoption-drain-time-s) 选项设置。

> **注意**
>
> 任何在 Envoy 配置中静态定义的监听器都不能通过 LDS API 进行修改或删除。

## 配置

- [v1 LDS API](https://www.envoyproxy.io/docs/envoy/latest/api-v1/listeners/lds.html#config-listeners-lds-v1)
- [v2 LDS API](../overview/v2_overview.md#v2-grpc-streaming-endpoints)

## 统计

LDS 的统计树是以 `listener_manager.lds` 为根，统计如下：

|	名称	|	类型	|	描述	|
| ---- | ---- | ---- |
|	config_reload	|	Counter	|	由于不同的配置更新，导致配置 API 调用总数	|
|	update_attempt	|	Counter	|	LDS 配置 API 调用重试总数	|
|	update_success	|	Counter	|	LDS 配置 API 调用成功总数	|
|	update_failure	|	Counter	|	LDS 配置 API 调用失败总数（网络或模式错误）	|
|	version	|	Gauge	|	上次成功调用的内容哈希值	|
