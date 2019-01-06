# 命令行选项

Envoy 由 JSON 配置文件和一组命令行选项一起驱动。以下是 Envoy 支持的命令行选项。

- `-c <path string>, --config-path <path string>`

  *(可选)* v1 或 v2 [JSON/YAML/proto3 配置文件](../configuration/configuration.md#config)的路径。 若未设置此选项，需要指定 [`--config-yaml`](#cmdoption-config-yaml) 选项。它会首先作为 [v2 引导配置文件](../configuration/overview/v2_overview.md#config-overview-v2-bootstrap)进行解析，若解析失败，会根据 [`--v2-config-only`] 选项决定是否作为 [v1 JSON 配置文件](../configuration/overview/v1_overview.md#config-overview-v1)进行解析。对于 v2 的配置文件，有效的扩展名包括 `.json`、 `.yaml`、 `.pb` 和 `.pb_text`，分别表示JSON、YAML、[二进制 proto3](https://developers.google.com/protocol-buffers/docs/encoding) 和 [文本 proto3](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.text_format) 格式。

- `--config-yaml <yaml string>`

  *(可选)* 一个 YAML 字符串，作为 v2 引导配置。若同时设置了 [`--config-path`](#cmdoption-c)，此 YAML 字符串的值会被合并到 [`--config-path`](#cmdoption-c) 指定的引导配置并覆盖相应选项。由于 YAML 是 JSON 的超集，也可以给 [`--config-yaml`](#cmdoption-config-yaml) 传入一个 JSON 字符串。 [`--config-yaml`](#cmdoption-config-yaml) 与 v1 引导配置不兼容。

  一个通过命令行覆盖节点 id 的例子：

   `./envoy -c bootstrap.yaml –config-yaml "node: {id: 'node1'}"`

- `--v2-config-only`

  *(可选)* 此标识决定配置文件是否应该只被解析为 [v2 引导配置文件](../configuration/overview/v2_overview.md#config-overview-v2-bootstrap)。若传入 false（默认选项），v2 引导配置解析失败时，会再次尝试将其解析为 [v1 JSON 配置文件](../configuration/overview/v1_overview.md#config-overview-v1)。

- `--mode <string>`

  *(可选)* Envoy 的执行模式之一：

  * `serve`: *（默认选项）* 校验 JSON 配置，然后正常提供服务。
  * `validate`：校验 JSON 配置，然后退出。会打印一条 “OK” 消息（若退出码为 0）或所有配置文件产生的错误（若退出码为 1）。不会产生网络流量，热重启流程也不会执行，所以不会影响到机器上其他的 Envoy 进程。

- `--admin-address-path <path string>`

  *(可选)* 输出管理地址和端口的文件路径。

- `--local-address-ip-version <string>`

  *(可选)* 填充服务器本地 IP 地址使用的 IP 地址版本。此参数影响各种头部信息，包括附加到 X-Forwarded-For（XFF）头部的内容。选项是 `v4` 或 `v6`。默认是 `v4`。

- `--base-id <integer>`

  *(可选)* 分配共享内存区域时使用的基本 ID。Envoy 在[热重启](../intro/arch_overview/hot_restart.md#arch-overview-hot-restart)时使用共享内存区域。大部分用户永远都不需要设置这个选项。不过，若需要在同一台机器上多次运行 Envoy，每个运行的 Envoy 需要一个唯一的基本 ID，以免共享内存区域产生冲突。

- `--concurrency <integer>`

  *(可选)* 启动的 [工作线程数](../intro/arch_overview/threading_model.md#arch-overview-threading)。若未指定则默认使用机器上的硬件线程数。

- `-l <string>, --log-level <string>`

  *(可选)* 日志级别。非开发者通常不应该设置这个选项。有关可用的日志级别和默认值，请参阅帮助文本。

- `--log-path <path string>`

  *(可选)* 输出日志的文件路径。处理 SIGUSR1 时，该文件将被重新打开。若未设置，输出到 stderr。

- `--log-format <format string>`

  *(可选)* 用于格式化日志消息元数据的格式字符串。若未设置，会使用一个默认的格式字符串 “[%Y-%m-%d %T.%e][%t][%l][%n] %v"。

支持的格式化标记有（包括示例输出）：

| 参数  | 解释 |
|---- | ---- |
| %v: | 要记录的实际消息 (“some user text”) |
|%t: | 线程 id (“1232”)|
|%P: | 进程 id (“3456”)|
|%n: | 记录器名称 (“filter”)|
|%l: | 消息的日志级别 （“debug”、 “info”） |
|%L: | 消息的日志级别缩写 (“D”、 “I”等) |
|%a: | 星期的缩写呈现 (“Tue”)|
|%A: | 星期的完整名称 (“Tuesday”)|
|%b: | 月份的缩写呈现 (“Mar”)|
|%B: | 月份的完整名称 (“March”)|
|%c: | 日期和时间的呈现 (“Tue Mar 27 15:25:06 2018”)|
|%C: | 年份的 2 位呈现 (“18”)|
|%Y: | 年份的 4 位呈现 (“2018”)|
|%D, %x: | 日期 MM/DD/YY 格式的缩写呈现 (“03/27/18”)|
|%m: | 月份 01-12 (“03”)|
|%d: | 月中的日期 01-31 (“27”)|
|%H: | 24 小时制的小时 00-23 (“15”)|
|%I: | 12 小时制的小时 01-12 (“03”)|
|%M: | 分钟 00-59 (“25”)|
|%S: | 秒数 00-59 (“06”)|
|%e: | 当前秒中的毫秒部分 000-999 (“008”)|
|%f: | 当前秒中的微秒部分 000000-999999 (“008789”)|
|%F: | 当前秒中的纳秒部分 000000000-999999999 (“008789123”)|
|%p: | 上午/下午 AM/PM (“AM”)|
|%r: | 12 小时制的钟表呈现 (“03:25:06 PM”)|
|%R: | 24 小时制 HH:MM 格式的时间, 等同于 %H:%M (“15:25”)|
|%T, %X: | ISO 8601 时间格式 (HH:MM:SS)，等同于 %H:%M:%S (“13:25:06”) |
|%z: | ISO 8601 与 UTC 的时区偏移 ([+/-]HH:MM) (“-07:00”)|
|%%: | % 符号 (“%”)|

- `--restart-epoch <integer>`

  *(可选)* [热重启](../intro/arch_overview/hot_restart.md#arch-overview-hot-restart)周期（Envoy 被热重启而不是全新启动的次数）。对于第一次启动默认为 0。此选项告诉 Envoy 是尝试创建，还是打开一个已存在的热重启所需的共享内存区域。每次热重启后它都应该被增加。多数情况下[热重启包装器](hot_restarter.md#operations-hot-restarter)设置的 *RESTART_EPOCH* 环境变量应该被传递给这个选项。

- `--hot-restart-version`

  *(可选)* 为当前的二进制文件输出一个热重启兼容性版本。可以将其与 [`GET /hot_restart_version`](admin.md#get--hot_restart_version) 管理接口的输出进行比较，以确定新的二进制文件和正在运行的二进制文件是否热重启兼容。

- `--service-cluster <string>`

  *(可选)* 定义 Envoy 运行的本地服务集群名称。本地服务集群名称首先来自[引导节点](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#envoy-api-field-config-bootstrap-v2-bootstrap-node)消息的[cluster](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/base.proto#envoy-api-field-core-node-cluster)字段。此命令行选项为指定此值提供了另一种方法，并将覆盖引导配置中设置的任何值。若使用了以下任何功能，则应该通过此命令行选项或引导配置来设置它：[statsd](../intro/arch_overview/statistics.md#arch-overview-statistics)、[健康检查集群验证](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_hc#config-cluster-manager-cluster-hc-service-name)、[运行时覆盖目录](https://www.envoyproxy.io/docs/envoy/latest/api-v1/runtime#config-runtime-override-subdirectory)、[User Agent 添加](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-add-user-agent)、 [HTTP 全局速率限制](../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit)、[CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) 和 [HTTP 跟踪](../intro/arch_overview/tracing.md#arch-overview-tracing)。

- `--service-node <string>`

  *(可选)* 定义 Envoy 运行的本地服务节点名称。本地服务节点名称首先来自[引导节点](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#envoy-api-field-config-bootstrap-v2-bootstrap-node)消息的消息的 id 字段。此命令行选项为指定此值提供了另一种方法，并将覆盖引导配置中设置的任何值。若使用了以下任何功能，则应该通过此命令行选项或引导配置来设置它：[statsd](../intro/arch_overview/statistics.md#arch-overview-statistics)、[CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) 和 [HTTP 跟踪](../intro/arch_overview/tracing.md#arch-overview-tracing)。

- `--service-zone <string>`

  *(可选)* 定义 Envoy 运行的本地服务区域名称。本地服务区域名称首先来自[引导节点](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#envoy-api-field-config-bootstrap-v2-bootstrap-node)消息的消息的 [locality.zone](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/base.proto#envoy-api-field-core-locality-zone) 字段。此命令行选项为指定此值提供了另一种方法，并将覆盖引导配置中设置的任何值。若使用了发现服务路由且发现服务暴露出[区域数据](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/sds#config-cluster-manager-sds-api-host-az)，则应该通过此命令行选项或引导配置来设置它。区域的含义是依赖于上下文的，如 AWS 上的[可用性区域（AZ ）](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)，GCP 上的[区域](https://cloud.google.com/compute/docs/regions-zones/)，等等。

- `--file-flush-interval-msec <integer>`

  *(可选)* 文件缓冲区刷新间隔（毫秒）。默认为 10 秒。此设置在文件创建期间用于确定缓冲区刷新到文件的间隔时间。缓冲区在每次写满时或每次间隔过后都会刷新，以先到者为准。调整此设置在跟踪输出[访问日志](../intro/arch_overview/access_logging.md#arch-overview-access-logs)时很有用，可以获得更多（或更少）的即时刷新。

- `--drain-time-s <integer>`

  *(可选)* 热重启期间 Envoy 将耗尽连接的时间（秒）。请参阅[热重启概述](../intro/arch_overview/hot_restart.md#arch-overview-hot-restart)了解更多信息。默认为 600 秒（10 分钟）。通常耗尽时间应小于通过 [`--parent-shutdown-time-s`](#cmdoption-parent-shutdown-time-s) 选项设置的父进程关闭时间。如何配置这两个设置取决于具体的部署。在边缘的场景下，可能需要耗费很长时间。在服务到服务的场景下，耗尽和关闭的时间可能缩短很多（例如，60s/90s）。

- `--parent-shutdown-time-s <integer>`

  *(可选)* Envoy 在热重启时关闭父进程之前等待的时间（秒）。请参阅[热重启概述](../intro/arch_overview/hot_restart.md#arch-overview-hot-restart)了解更多信息。默认为 900 秒（15 分钟）。

- `--max-obj-name-len <uint64_t>`

  *(可选)* cluster/route_config/listener 中名称字段的最大长度（以字节为单位）。此选项通常用于自动生成集群名称的场景，通常超过会 60 字符的内部限制。默认为 60。

  > *注意：此设置会影响 [`--hot-restart-version`](#cmdoption-hot-restart-version) 的输出。若您开始使用此选项并设置为非默认值，则应该使用相同的值配置到热重启的新进程。*

- `--max-stats <uint64_t>`

  *(可选)* 热重启间可以共享统计的最大数量。此设置会影响 [`--hot-restart-version`](#cmdoption-hot-restart-version) 的输出；热重启必须使用相同的值。默认为 16384。

- `--disable-hot-restart`

  *(可选)* 此标识禁用已启用热重启的 Envoy 版本的热重启。 默认情况下，热重启是启用的。
