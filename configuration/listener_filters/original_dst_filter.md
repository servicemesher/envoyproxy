# 原始目的地

原始目的地监听器过滤器读取 SO_ORIGINAL_DST 套接字属性，这个属性在连接被重定向时设置。重定向可以通过 `iptables REDIRECT target` 或者  `iptables TPROXY target` 实现，结合使用监听器的 [transparent](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#envoy-api-field-listener-transparent) 属性设置。Envoy 中的后续处理将恢复后的目标地址视为连接的本地地址，而不是监听器正在监听的地址。此外， [原始目标集群](../../intro/arch_overview/service_discovery.md#arch-overview-service-discovery-types-original-destination)可用于将 HTTP 请求或 TCP 连接转发到恢复后的目标地址。

- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto.html#envoy-api-field-listener-filter-name)