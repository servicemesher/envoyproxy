# HTTP路由

Envoy 包括一个 HTTP [路由器过滤器](https://github.com/servicemesher/envoy/blob/master/configuration/http_filters/router_filter.md#config-http-filters-router)，它可以被安装来执行高级路由任务。这对于处理边缘流量（传统的反向代理请求处理）以及为服务 Envoy  网格构建服务（通常是通过主机/授权 HTTP 头的路由到达特定的上游服务集群）非常有用。Envoy 也可以被配置为转发代理。在正向代理配置中，网格客户端可以通过适当地配置他们的 http 代理来作为 Envoy。在高级层由接受一个传入的 HTTP 请求，将其与上游集群相匹配，在上游集群中获得一个[连接池](https://github.com/servicemesher/envoy/blob/master/intro/arch_overview/connection_pooling.md#arch-overview-conn-pool)，并转发请求。路由过滤器支持以下特性：

- 将域/授权映射到一组路由规则的虚拟主机。
- 前缀和精确路径匹配规则(都有[敏感](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-case-sensitive)和不敏感)。目前还不支持正则/slug 匹配，这主要是因为以编程方式确定路由规则是否相互冲突变得非常困难,然而，我们可能会根据需求增加对未来的支持。
- 在虚拟主机级别上的 [TLS 重定向](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/vhost.md#config-http-conn-man-route-table-vhost-require-ssl)。
- 在路由级别的[路径/主机](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/vhost.md#config-http-conn-man-route-table-vhost-require-ssl)重定向。
- 在路由级别的直接（非代理）HTTP 响应。
- [显式重写](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-host-rewrite)。
- [自动主机重写](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-auto-host-rewrite)，基于所选的上游主机的 DNS 名称。
- [前缀重写](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-prefix-rewrite)。
- 在路由级别上的 [Websocket升级](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-use-websocket)。
- 请求通过 HTTP 头或路径配置[指定的重试](https://github.com/servicemesher/envoy/blob/master/intro/arch_overview/http_routing.md#arch-overview-http-routing-retry)。
- 请求通过 [HTTP头](https://github.com/servicemesher/envoy/blob/master/configuration/http_filters/router_filter.md#config-http-filters-router-headers) 或[路径配置](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-timeout)指定超时。
- 通过[运行时值](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-runtime)从一个上游集群转移到另一个集群（参见[流量转移/分割](https://github.com/servicemesher/envoy/blob/master/configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting)）。
- 使用基于[权重/百分比的路由](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-weighted-clusters)的流量跨越多个上游集群（参见[流量转移/分割](https://github.com/servicemesher/envoy/blob/master/configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting-split)）。
- 任意标题匹配[路由规则](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-headers)。
- 虚拟集群规范。虚拟集群是在虚拟主机级别上指定的，由 Envoy  使用，在标准集群级别上生成额外的统计信息。虚拟集群可以使用正则表达式匹配。
- 基于[优先级](https://github.com/servicemesher/envoy/blob/master/intro/arch_overview/http_routing.md#arch-overview-http-routing-priority)的路由。
- 基于[哈希](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-hash-policy)策略的路由。
- [绝对url ](https://github.com/servicemesher/envoy/blob/master/api-v1/network_filters/http_conn_man.md#config-http-conn-man-http1-settings)支持非 tls 转发代理。


## 路由表

HTTP 连接管理器的配置拥有所有配置的 HTTP 过滤器所使用的[路由表](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route_config.md#config-http-conn-man-route-table)。尽管路由器过滤器是路由表的主要使用者，但是其他过滤器也有访问权限，以防它们想根据请求的最终目的地做出决定。例如，内置的速率限制过滤器查询路由表，以确定是否应该根据路由调用全局速率限制服务。连接管理器确保对特定请求的所有调用都是稳定的，即使决策涉及到随机性（例如，在运行时配置路由规则的情况下）。

## 重试语义

Envoy 允许在[路由配置](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route-retry)和通过[请求头](https://github.com/servicemesher/envoy/blob/master/configuration/http_filters/router_filter.md#config-http-filters-router-headers)的特定请求中配置重试。以下配置是可能的：

- **最大数量的重试**:Envoy 将继续进行多次尝试。在每次重试之间使用一个指数回溯算法。此外，*所有重试都包含在总体请求超时中*。这避免了由于大量重试而导致的长时间请求。
- **重试条件**：Envoy 可以根据应用程序的要求对不同类型的条件进行重试。例如，网络故障，所有5xx响应代码，幂等4xx响应码，等等。
 

请注意，根据 [x-envoy-overloaded](https://github.com/servicemesher/envoy/blob/master/configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-overloaded)的内容，重试可能会被禁用。

## 优先级路由

Envoy 支持[路由](https://github.com/servicemesher/envoy/blob/master/api-v1/route_config/route.md#config-http-conn-man-route-table-route)级别的优先路由。当前的优先级实现为每个优先级使用不同的[连接池](https://github.com/servicemesher/envoy/blob/master/intro/arch_overview/connection_pooling.md#arch-overview-conn-pool)和[断路设置](https://github.com/servicemesher/envoy/blob/master/configuration/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers)。这意味着，即使对于 http/2 请求，也将使用两个物理连接到上游主机。在未来，特使可能会支持真正的 http/2 优先级而不是单一连接。

当前支持的优先级是*默认的*和*高的*。

## 直接响应

Envoy 支持发送“直接”回应。这些是预先配置的 HTTP 响应，不需要代理到上游服务器。

有两种方法可以在一条路由中指定直接响应：
- 设置 [direct_response](https://github.com/servicemesher/envoy/blob/master/api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-direct-response)字段。这适用于所有 HTTP 响应状态。
- 设置[重定向](https://github.com/servicemesher/envoy/blob/master/api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-redirect)。这只适用于重定向响应状态，但它简化了*位置*头的设置。

直接响应有一个 HTTP 状态码和一个可选体。路由配置可以内联地指定响应体，或者指定包含体的文件的路径名。如果路线配置指定了一个文件路径名，Envoy  将在配置负载上读取文件并缓存内容。

注意

如果指定了响应体，那么它的大小必须不超过4 KB ，不管它是内联还是在文件中。Envoy 目前将整体保存在内存中，因此4 KB的限制是为了防止代理的内存占用太大。

如果**应答器添加**被设置为路由或封闭虚拟主机，则Envoy 将在直接 HTTP 响应中包含指定的头部。