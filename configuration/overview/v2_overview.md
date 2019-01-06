# v2 API 概览

Envoy v2 API 被定义为 [data plane API repository](https://github.com/envoyproxy/data-plane-api/) 中的 [proto3](https://developers.google.com/protocol-buffers/docs/proto3) [Protocol Buffers](https://developers.google.com/protocol-buffers/)。其改进了现有的 [v1 API 和概念](v1_overview.html#config-overview-v1)以支持：

- 通过 gRPC 流式传输对 [xDS](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md) API 的更新，这减少了资源的需求并且可以降低更新延迟。
- 一种新的 REST-JSON API。其中 JSON/YAML 格式是通过 [proto3 规范的 JSON 映射](https://developers.google.com/protocol-buffers/docs/proto3#json)机制派生出来的。  
- 通过文件系统、REST-JSON 或 gRPC 端点传递更新。  
- 通过扩展端点分配 API 进行高级负载均衡，并向管理服务器报告负载以及资源的利用率。
- 当需要[更强的一致性和排序属性](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md#eventual-consistency-considerations)时，Envoy v2 API 仍然可以保持基准最终一致性模型。  

Envoy 与管理服务器之间的关于 v2 消息交换方面的更多详细信息，请参阅 [xDS 协议说明](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md)。 

## Bootstrap 配置

要使用 v2 API，需要提供引导程序配置文件。其提供了静态服务器配置以及根据需要配置 Envoy 以访问[动态配置](../../intro/arch_overview/dynamic_configuration.html#arch-overview-dynamic-config)。 与 v1 JSON/YAML 配置一样，可以在命令行通过[`-c`](../../operations/cli.html#cmdoption-c) 标志提供，即：

```bash
./envoy -c <path to config>.{json,yaml,pb,pb_text} --v2-config-only
```

扩展映射底层 v2 配置。 [`--v2-config-only`](../../operations/cli.html#cmdoption-v2-config-only) 标志并不是严格要求的，因为 Envoy 会自动的尝试监测配置文件的版本，但是在配置解析失败时，该选项可以提供增强的调试体验。  

[Bootstrap](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) 消息是配置的根本来源，[Bootstrap](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) 消息中一个关键的概念是静态和动态资源的之间的区别。例如 [Listener](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto.html#envoy-api-msg-listener) 或 [Cluster](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto.html#envoy-api-msg-cluster) 这些资源既可以从 [static_resources](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-static-resources) 静态的获得也可以从 [dynamic_resources](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) 中配置的 [LDS](../listeners/lds.html#config-listeners-lds) 或 [CDS](../cluster_manager/cds.html#config-cluster-manager-cds) 之类的 xDS 服务获取。  

## 示例

下面我们将使用 `YAML` 表示的配置原型，以及从127.0.0.1:10000 到 127.0.0.2:1234 的服务代理 HTTP 的运行示例。  

### 静态

下面提供了一个最小的完全静态引导配置：

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 127.0.0.1, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: some_service }
          http_filters:
          - name: envoy.router
  clusters:
  - name: some_service
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    hosts: [{ socket_address: { address: 127.0.0.2, port_value: 1234 }}]
```

### 除了 EDS 是动态的其他大部分为静态

下面提供了一个 bootstrap 配置，该配置从以上示例开始，通过监听 127.0.0.3:5678 的 [EDS](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/eds.proto.html#envoy-api-file-envoy-api-v2-eds-proto) gRPC管理服务器进行[动态端点发现](../../intro/arch_overview/dynamic_configuration.html#arch-overview-dynamic-config-sds)：

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 127.0.0.1, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: some_service }
          http_filters:
          - name: envoy.router
  clusters:
  - name: some_service
    connect_timeout: 0.25s
    lb_policy: ROUND_ROBIN
    type: EDS
    eds_cluster_config:
      eds_config:
        api_config_source:
          api_type: GRPC
          cluster_names: [xds_cluster]
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    hosts: [{ socket_address: { address: 127.0.0.3, port_value: 5678 }}]
```

注意上面 *xds_cluster* 被定义为指向 Envoy 管理服务器。即使在完全动态的配置中，也需要定义一些静态资源，从而将 Envoy 指向其 xDS 管理服务器。  

在上面的例子中，EDS 管理服务器可以返回一个 proto 编码的 [DiscoveryResponse](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/discovery.proto.html#envoy-api-msg-discoveryresponse)：

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.ClusterLoadAssignment
  cluster_name: some_service
  endpoints:
  - lb_endpoints:
    - endpoint:
        address:
          socket_address:
            address: 127.0.0.2
            port_value: 1234
```

上面显示的版本控制和类型 URL 方案在[流式 gRPC 订阅协议](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md#streaming-grpc-subscriptions)文档中有更详细的解释。  

### 动态

下面提供了完全动态的 `bootstrap` 配置，其中属于管理服务器的所有资源都是通过 xDS 发现的:  

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }

dynamic_resources:
  lds_config:
    api_config_source:
      api_type: GRPC
      cluster_names: [xds_cluster]
  cds_config:
    api_config_source:
      api_type: GRPC
      cluster_names: [xds_cluster]

static_resources:
  clusters:
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    hosts: [{ socket_address: { address: 127.0.0.3, port_value: 5678 }}]
```

管理服务器将如下响应 `LDS` 请求:  

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.Listener
  name: listener_0
  address:
    socket_address:
      address: 127.0.0.1
      port_value: 10000
  filter_chains:
  - filters:
    - name: envoy.http_connection_manager
      config:
        stat_prefix: ingress_http
        codec_type: AUTO
        rds:
          route_config_name: local_route
          config_source:
            api_config_source:
              api_type: GRPC
              cluster_names: [xds_cluster]
        http_filters:
        - name: envoy.router
```

管理服务将如下响应 `RDS` 请求：  

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.RouteConfiguration
  name: local_route
  virtual_hosts:
  - name: local_service
    domains: ["*"]
    routes:
    - match: { prefix: "/" }
      route: { cluster: some_service }
```

管理服务器将如下响应 `CDS` 请求：

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.Cluster
  name: some_service
  connect_timeout: 0.25s
  lb_policy: ROUND_ROBIN
  type: EDS
  eds_cluster_config:
    eds_config:
      api_config_source:
        api_type: GRPC
        cluster_names: [xds_cluster]
```

管理服务器将如下响应 `EDS` 请求：

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.ClusterLoadAssignment
  cluster_name: some_service
  endpoints:
  - lb_endpoints:
    - endpoint:
        address:
          socket_address:
            address: 127.0.0.2
            port_value: 1234
```

## 管理服务器

v2 xDS 管理服务器将按照 `gRPC` 和/或 `REST` 服务的要求实现以下端点。在流式 `gRPC` 和 `REST-JSON` 两种情况下，都会发送 [`DiscoveryRequest`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/discovery.proto.html#envoy-api-msg-discoveryrequest) 并根据 [xDS 协议](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md)接收 `DiscoveryResponse`。  

### gRPC streaming 端点

- `POST /envoy.api.v2.ClusterDiscoveryService/StreamClusters`


有关服务定义，请参考 [`cds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/cds.proto)。当在 [`Bootstrap`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) 配置的 [dynamic_resources](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) 中设置为

```yaml
cds_config:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

时 Envoy 将此用作客户端。  

- `POST /envoy.api.v2.EndpointDiscoveryService/StreamEndpoints`


有关服务定义，请参阅 [`eds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/eds.proto)。在 [`Cluster`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto.html#envoy-api-msg-cluster) 配置的 [`eds_cluster_config`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto.html#envoy-api-field-cluster-eds-cluster-config) 字段中设置为   


```yaml
eds_config:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

时，Envoy 将此用作客户端。  

- `POST /envoy.api.v2.ListenerDiscoveryService/StreamListeners`


有关服务定义，请参阅 [`lds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/lds.proto)。 当在 [`Bootstrap`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) 配置的 [`dynamic_resources`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) 中设置为  


```yaml
lds_config:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

时，Envoy将此用作客户端。

- `POST /envoy.api.v2.RouteDiscoveryService/StreamRoutes`


有关服务定义，请参阅 [`rds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/rds.proto)。 当在 [HttpConnectionManager](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager) 配置的 [rds](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-rds) 字段中设置为   

```yaml
route_config_name: some_route_name
config_source:
  api_config_source:
    api_type: GRPC
    cluster_names: [some_xds_cluster]
```

时，Envoy 将它用作客户端。   

### REST 端点

- `POST /v2/discovery:clusters`

有关服务定义，请参阅 [`cds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/cds.proto)。 当在 [`Bootstrap`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) 配置的 `dynamic_resources`中设置
为  

```yaml
cds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

时，Envoy 将此用作客户端。    


- `POST /v2/discovery:endpoints`


有关服务定义，请参阅 [`eds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/eds.proto)。在 [`Cluster`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto.html#envoy-api-msg-cluster) 配置的 [`eds_cluster_config`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/eds.proto) 字段中设置为

```yaml
eds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

时，Envoy 将此用作客户端。  
- `POST /v2/discovery:listeners`


有关服务定义，请参阅 [`lds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/lds.proto)。 当在 [`Bootstrap`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) 配置的 [`dynamic_resources`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) 中设置为

```yaml
lds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

时，Envoy 将此用作客户端。  

- `POST /v2/discovery:routes`


有关服务定义，请参阅 [`rds.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/rds.proto)。 当在[`HttpConnectionManager`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager) 配置的[`rds`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-rds)字段中设置为    

```yaml
route_config_name: some_route_name
config_source:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

时，Envoy 将它用作客户端。  

## 聚合发现服务

虽然 Envoy 从根本上采用了最终的一致性模型，但 ADS 提供了对 API 更新推送进行排序并确保单个管理服务器与 Envoy 节点进行 API 更新相关性的机会。ADS 允许管理服务器在一个单一的双向 gRPC 流上传递一个或多个 API 及其资源。没有这些，一些如 RDS 和 EDS 的 API 就可能需要管理多个流并连接到不同的管理服务器。  

`ADS` 将允许通过适当的排序无损的更新配置。例如，假设 *foo.com* 映射到集群 *X*。我们希望将路由表中的映射更改为集群 *Y* 中的 *foo.com*。为了做到这一点，必须首先提供包含两个集群 *X* 和 *Y* 的 `CDS/EDS` 更新。  

如果没有 `ADS`，`CDS/EDS/RDS` 流可能指向不同的管理服务器，或者位于同一管理服务器上的不同 `gRPC` 流和连接需要协调。 `EDS` 资源请求可以分成两个不同的流，一个用于 *X*，另一个用于 *Y*。`ADS`允许将这些请求合并为单个流到单个管理服务器，避免了分布式同步的需要，以正确地对更新进行排序。依靠 `ADS`，管理服务器将在单个数据流上提供`CDS`，`EDS`和`RDS`更新。  

ADS 仅适用于 gRPC 流媒体（不是REST），在[本文档](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md#aggregated-discovery-services-ads)中有更详细的描述。`gRPC` 的端点是:   

- `POST /envoy.api.v2.AggregatedDiscoveryService/StreamAggregatedResources`

有关服务定义，请参阅 [`discovery.proto`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/discovery.proto)。 当在 [`Bootstrap`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-msg-config-bootstrap-v2-bootstrap) 配置的 [`dynamic_resources`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto.html#envoy-api-field-config-bootstrap-v2-bootstrap-dynamic-resources) 中设置。

在 [discovery.proto](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/discovery.proto) 查看服务配置。当以下用于 Envoy 的客户端配置为

```yaml
ads_config:
  api_type: GRPC
  cluster_names: [some_ads_cluster]
```

时，Envoy 将此用作客户端。  

设置此项时，可以将[上述](#v2-grpc-streaming-endpoints)任何配置源设置为使用ADS通道。 例如，LDS 配置可以从


```yaml
lds_config:
  api_config_source:
    api_type: REST
    cluster_names: [some_xds_cluster]
```

更改为  

```yaml
lds_config: {ads: {}}
```

其效果是 LDS 流将通过共享 ADS 通道指向 *some_ads_cluster*。

## 管理服务器不可达

当 `Envoy` 实例失去与管理服务器的连接时，`Envoy` 将锁定到先前的配置，同时在后台主动重试以重新建立与管理服务器的连接。  

Envoy 调试记录了每次尝试连接时都无法与管理服务器建立连接的事实。  

[upstream_cx_connect_fail](../cluster_manager/cluster_stats.html#config-cluster-manager-cluster-stats) 指向管理服务器的集群等级统计信息提供了用于监视此行为的信号。  

## 状态

除非另有说明，否则将实现 [v2 API参考文档](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api#envoy-api-reference)中描述的所有功能。 在 v2 API 参考文档和 [v2 API资源库](https://github.com/envoyproxy/data-plane-api/tree/master)中，所有原型都被*冻结*，除非它们被标记为*草稿*或*实验*。 在这里，冻结意味着我们不会破坏线格式兼容性。  

*Frozen* 原型可以进一步的扩充。例如：通过添加新的字段，以不破坏向[后兼容性](https://developers.google.com/protocol-buffers/docs/overview#how-do-they-work)的方式。上述原型中的字段可能会在不再使用相关功能的情况下，随着[违反变更策略](https://github.com/envoyproxy/envoy/blob/master//CONTRIBUTING.md#breaking-change-policy)而被弃用。 尽管 *Frozen* 的 API 保持其格式兼容性，但是保留了更改原名称空间、文件位置以及嵌套关系的权利，这可能会导致代码更改中断。 我们的目标是尽量减少流失。  


标记为 *draft* 的原型以为这已经接近完成, 可能至少部分在 Envoy 中实现，但可能会在冻结之前破坏线路格式。  

标记为 *experimental* 的原型与原始草案有相同的警告，并可能在执行和冻结之前做出重大更改。  

[这里](https://github.com/envoyproxy/envoy/issues?q=is%3Aopen+is%3Aissue+label%3A%22v2+API%22)可以跟踪当前开放的 v2 API 问题。  
