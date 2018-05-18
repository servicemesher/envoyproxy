# 文件系统标志

Envoy 支持文件系统 "标志"在启动之后改变状态。在需要的情况下，该功能用于保持重启之间的变化。标志文件应该被放置在 [flags_path](../configuration/overview/v1_overview.md#config-overview-flags-path) 配置选项指定的目录中。当前支持的标志文件是：

- drain

  如果这个文件存在，Envoy 将以 HC 失败模式启动，类似于 [`POST /healthcheck/fail`](admin.md#post--healthcheck-fail)命令被执行之后。

