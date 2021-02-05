# 统计

- [概要](#general)
- [健康检查统计](#health-check-statistics)
- [异常点检测统计](#outlier-detection-statistics)
- [动态 HTTP 统计](#dynamic-http-statistics)
- [可变树动态 HTTP 统计](#alternate-tree-dynamic-http-statistics)
- [每个服务区域动态 HTTP 统计](#per-service-zone-dynamic-http-statistics)
- [负载均衡统计](#load-balancer-statistics)
- [负载均衡子集统计](#load-balancer-subset-statistics)

## 概要

集群管理器有一个以  *cluster_manager.*  为根的统计树。有以下统计项。统计名称中的任何`:` 字符将会被替换成`_`。

| 名称          | 类型 | 描述        |
| ---------------- | ------- | ------------------------------------------------------ |
| cluster_added | Counter | 添加的所有集群数 （通过静态配置或 CDS） |
| cluster_modified | Counter | 修改的所有集群数 （通过 CDS）      |
| cluster_removed  | Counter | 移除的所有集群数 （通过 CDS） |
| active_clusters  | Gauge| 当前活跃的（预热的）所有集群数 |
| warming_clusters | Gauge| 当前正在预热的活动（非活跃的）所有集群数 |

每一个集群有一个以 *cluster.<name>.* 为根的统计树。有以下统计项:

| 名称          | 类型 | 描述        |
| ---------------- | ------- | ------------------------------------------------------ |
| upstream_cx_total                        | Counter | 总连接数      |
| upstream_cx_active                       | Gauge | 总激活的连接数 |
| upstream_cx_http1_total                  | Counter | HTTP/1.1 总连接数 |
| upstream_cx_http2_total                  | Counter | Total HTTP/2 总连接数                                   |
| upstream_cx_connect_fail                 | Counter | 总连接失败数    |
| upstream_cx_connect_timeout              | Counter | 总连接超时数    |
| upstream_cx_idle_timeout                 | Counter | 总连接空闲超时  |
| upstream_cx_connect_attempts_exceeded    | Counter | 超过配置连接尝试的总连续连接失败数                      |
| upstream_cx_overflow                     | Counter | 机群连接断路器溢出的总次数                              |
| upstream_cx_connect_ms                   | Histogram | 连接建立毫秒    |
| upstream_cx_length_ms                    | Histogram | 连接长度毫秒    |
| upstream_cx_destroy                      | Counter | 完全破坏连接    |
| upstream_cx_destroy_local                | Counter | 本地连接破坏    |
| upstream_cx_destroy_remote               | Counter | 远程连接完全销毁数 |
| upstream_cx_destroy_with_active_rq       | Counter | 用1 +主动请求销毁总连接数                               |
| upstream_cx_destroy_local_with_active_rq | Counter | 用1 +主动请求销毁本地总连接数                           |
| upstream_cx_destroy_remote_with_active_rq| Counter | 用1 +主动请求远程销毁总连接                             |
| upstream_cx_close_notify                 | Counter | 通过 HTTP/1.1 连接关闭报头或HTTP/2 GOAWAY 关闭的总连接数 |
| upstream_cx_rx_bytes_total               | Counter | 接收到的总连接字节数                                    |
| upstream_cx_rx_bytes_buffered            | Gauge | 当前缓冲的接收连接字节                                  |
| upstream_cx_tx_bytes_total               | Counter | 发送的连接字节总数 |
| upstream_cx_tx_bytes_buffered            | Gauge | 发送当前缓冲的连接字节                                  |
| upstream_cx_protocol_error               | Counter | 总连接协议错误  |
| upstream_cx_max_requests                 | Counter | 由于最大请求关闭总连接                                  |
| upstream_cx_none_healthy                 | Counter | 由于没有健康主机，连接没有建立的总次数                  |
| upstream_rq_total                        | Counter | 总请求量        |
| upstream_rq_active                       | Gauge | 总活动请求    |
| upstream_rq_pending_total                | Counter | 等待连接池连接的总请求                                  |
| upstream_rq_pending_overflow             | Counter | 溢出连接池电路中断和失败的总请求                        |
| upstream_rq_pending_failure_eject        | Counter | 由于连接池连接失败而导致的总请求失败                    |
| upstream_rq_pending_active               | Gauge | 挂起连接池连接的全部活动请求                            |
| upstream_rq_cancelled                    | Counter | 在获得连接池连接之前取消的总请求                        |
| upstream_rq_maintenance_mode             | Counter | 由于维护模式导致的请求总数为 503                        |
| upstream_rq_timeout                      | Counter | 等待响应的总请求|
| upstream_rq_per_try_timeout              | Counter | 每次尝试超时的总请求                                    |
| upstream_rq_rx_reset                     | Counter | 远程重置的总请求|
| upstream_rq_tx_reset                     | Counter | 本地重置的总请求|
| upstream_rq_retry                        | Counter | 总请求重试      |
| upstream_rq_retry_success                | Counter | 总请求重试成功率|
| upstream_rq_retry_overflow               | Counter | 由于电路断开而未重试的总请求                            |
| upstream_flow_control_paused_reading_total  | Counter | 上游流量控制暂停读取的总次数                            |
| upstream_flow_control_resumed_reading_total | Counter | 从上游读取的流量控制的总次数                            |
| upstream_flow_control_backed_up_total    | Counter | 上游连接备份和暂停读取的总次数从下游读取                |
| upstream_flow_control_drained_total      | Counter | 从上游读取和恢复上游连接的总次数                        |
| membership_change                        | Counter | 总簇成员变化    |
| membership_healthy                       | Gauge | 当前集群健康总量（包括健康检查和异常值检测）            |
| membership_total                         | Gauge | 当前群集成员总数|
| retry_or_shadow_abandoned                | Counter | 由于缓冲区限制，阴影或重试缓冲的总次数被取消。          |
| config_reload                            | Counter | 由于配置不同，导致的 API 重新加载导致配置重新加载。     |
| update_attempt                           | Counter | 总簇成员更新尝试|
| update_success                           | Counter | 总簇成员更新成功率 |
| update_failure                           | Counter | 总簇成员更新失败|
| update_empty                             | Counter | 使用空集群负载分配结束的总群集成员更新，并继续使用以前配置 |
| update_no_rebuild                        | Counter | 没有导致任何集群负载均衡结构重建的完全成功的群集成员更新|
| version                                  | Gauge | 从最后一次成功的 API 获取内容的哈希                     |
| max_host_weight                          | Gauge | 集群中任意主机的最大权重                                |
| bind_errors                              | Counter | 将套接字绑定到已配置的源地址的总错误                    |

## 健康检查统计

如果配置了健康检查，则该集群具有根于根 *cluster.<name>.health_check.* 的附加统计树。有如下统计:

| 名称          | 类型 | 描述        |
| ---------------- | ------- | ------------------------------------------------------ |
| attempt      | Counter | 健康检查次数                              |
| success      | Counter | 健康检查成功次数                          |
| failure      | Counter | 立即失效的健康检查（例如HTTP 503）以及网络故障的次数|
| passive_failure | Counter | 由于被动事件引起的健康检查失败的次数（例如X-Enviv-即时健康检查失败） |
| network_failure | Counter | 网络错误引起的健康检查失败次数                |
| verify_cluster  | Counter | 尝试群集名称验证的健康检查数量                |
| healthy      | Gauge | 健康会员人数                               |

## 异常点检测统计

如果为集群配置了[异常点检测](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/outlier#arch-overview-outlier-detection)，统计将以 *cluster.<name>.outlier_detection.* 为根，包含如下:

| 名称          | 类型 | 描述        |
| ---------------- | ------- | ------------------------------------------------------ |
| ejections_enforced_total                       | Counter | 由于异常值类型强制执行的驱逐次数                   |
| ejections_active                               | Gauge | 当前驱逐主机的数量                                 |
| ejections_overflow                             | Counter | 由于最大驱逐率而中止的驱逐数                       |
| ejections_enforced_consecutive_5xx             | Counter | 强制执行的连续 5xx 驱逐数                          |
| ejections_detected_consecutive_5xx             | Counter | 检测到的连续 5xx 驱逐数（即使未强制执行）          |
| ejections_enforced_success_rate                | Counter | 强制成功率异常离群数                               |
| ejections_detected_success_rate                | Counter | 检测成功率离群值的数量（即使未执行）               |
| ejections_enforced_consecutive_gateway_failure | Counter | 强制连续网关失败驱逐数                             |
| ejections_detected_consecutive_gateway_failure | Counter | 检测到的连续网关故障驱逐数目（即使未强制执行）     |
| ejections_total                                | Counter | 不赞成的由于异常值类型引起的驱逐次数（即使未执行） |
| ejections_consecutive_5xx                      | Counter | 不赞成的连续5xx驱逐数（即使未强制执行）            |

## 动态 HTTP 统计

如果使用 HTTP，动态 HTTP 响应代码统计也是可用的。 这些是由各种内部系统发出的，以及一些过滤器，如[路由器过滤器](https://www.envoyproxy.io/docs/envoy/latest/configuration/http_filters/router_filter)和[速率限制过滤器](https://www.envoyproxy.io/docs/envoy/latest/configuration/http_filters/rate_limit_filter)。统计将以 *cluster.<name>.* 为根，包含如下:

| 名称                    | 类型   | 描述      |
| -------------------------- | --------- | ---------------------------------------------------- |
| upstream_rq_<*xx>          | Counter | HTTP响应代码汇总（例如，2xx 3xx，等。）    |
| upstream_rq_<*>            | Counter | 特异性的HTTP响应代码（例如，201、302等。） |
| upstream_rq_time           | Histogram | 请求时间ms|
| canary.upstream_rq_<*xx>   | Counter | 上游的 canary 聚合 HTTP 响应代码 |
| canary.upstream_rq_<*>     | Counter | 上游的 canary HTTP 特定响应代码 |
| canary.upstream_rq_time    | Histogram | 上游的 canary 请求时间 ms |
| internal.upstream_rq_<*xx> | Counter | 内部原始聚合 HTTP 响应代码 |
| internal.upstream_rq_<*>   | Counter | 内部原始特定 HTTP 响应代码 |
| internal.upstream_rq_time  | Histogram | 内部原始请求时间 ms |
| external.upstream_rq_<*xx> | Counter | HTTP响应代码聚集外部性|
| external.upstream_rq_<*>   | Counter | HTTP响应代码和特异性|
| external.upstream_rq_time  | Histogram | 外部原始请求时间 ms |

## 可变树动态 HTTP 统计

如果配置了可变树统计信息，则它们将存在于 *cluster.<name>.<alt name>.*命名空间。所产生的统计数据与动态 HTTP 统计部分[以上](https://www.envoyproxy.io/docs/envoy/latest/configuration/cluster_manager/cluster_stats.html)所记录的数据相同。

## 每个服务区域动态 HTTP 统计

如果服务区域可用于本地服务 (通过 [`--service-zone`](https://www.envoyproxy.io/docs/envoy/latest/operations/cli#cmdoption-service-zone)) 和 [上游集群](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/service_discovery#arch-overview-service-discovery-types-sds)， Envoy 在将跟踪 *cluster.<name>.zone.<from_zone>.<to_zone>.* 命名空间的以下统计数据.

| 名称                    | 类型   | 描述  |
| ----------------- | --------- | ---------------------------------------------------- |
| upstream_rq_<*xx> | Counter| 聚合 HTTP 响应代码（例如，2xx 3xx，等。） |
| upstream_rq_<*>| Counter| 特定的 HTTP 响应代码（例如，201 等） |
| upstream_rq_time  | Histogram | 请求时间 ms                |

## 负载均衡统计

负载均衡器决策监测统计。统计以 *cluster.<name>.* 为根，包含如下统计: 

| 名称          | 类型 | 描述        |
| ---------------- | ------- | ------------------------------------------------------ |
| lb_recalculate_zone_structures | Counter | 重新生成局部感知路由结构的次数，用于上游位置选择的快速决策 |
| lb_healthy_panic               | Counter | 在恐慌模式下与负载平衡器平衡的总请求负载                   |
| lb_zone_cluster_too_small      | Counter | 由于上游簇大小小，无区域感知路由|
| lb_zone_routing_all_directly   | Counter | 将所有请求直接发送到同一区域   |
| lb_zone_routing_sampled        | Counter | 向同一区域发送一些请求        |
| lb_zone_routing_cross_zone     | Counter | 区域感知路由模式，但必须发送跨区域|
| lb_local_cluster_not_ok        | Counter | 本地主机集未设置或是本地集群的恐慌模式|
| lb_zone_number_differs         | Counter | 本地和上游集群的区域数目不同   |
| lb_zone_no_capacity_left       | Counter | 舍入误差导致随机区域选择结束的次数|

## 负载均衡子集统计

负载均衡器子集 <arch_overview_load_balancer_subsets> 决策的统计监测。统计以 *cluster.<name>.* 为根，包含如下统计:

| 名称                    | 类型   | 描述      |
| ------------------- | ------- | ---------------------------------------------------------- |
| lb_subsets_active   | Gauge   | 当前可用子集的数目          |
| lb_subsets_created  | Counter | 创建的子集数               |
| lb_subsets_removed  | Counter | 由于没有主机而删除的子集数   |
| lb_subsets_selected | Counter | 选择任何子集的负载平衡次数   |
| lb_subsets_fallback | Counter | 调用回退策略的次数          |