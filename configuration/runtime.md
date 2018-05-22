# 运行时

[runtime configuration](../intro/arch_overview/runtime.md#arch-overview-runtime)指定了包含可以重载配置元素的本地文件系统数的位置。值可以在[/runtime admin endpoint](../operations/admin.md#operations-admin-interface-runtime)查看。值可以在[/runtime_modify admin endpoint](../operations/admin.md#operations-admin-interface-runtime-modify)修改和追加. 如果没用进行运行时配置，则会使用空提供程序，该提供程序会使用代码中内置的除了通过/ runtime_modify添加的值之外的所有缺省值，。

注意

要小心使用[/runtime_modify](../operations/admin.md#operations-admin-interface-runtime-modify) 端点。 其变更是立即生效的。管理接口的[妥善的保护](../operations/admin.md#operations-admin-interface-security)是非常**重要**的。

- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/runtime#config-runtime-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#envoy-api-msg-config-bootstrap-v2-runtime)

## 文件系统设计

配置指南的各个部分描述了可用的运行时设置。 例如, [这里](cluster_manager/cluster_runtime.md#config-cluster-manager-cluster-runtime)是上游集群的运行时配置。

假定文件夹 `/srv/runtime/v1` 指向存储全局运行时配置的实际文件系统路径。以下是运行时的典型配置参数:

- *symlink_root*: `/srv/runtime/current`
- *subdirectory*: `envoy`
- *override_subdirectory*: `envoy_override`

这里`/srv/runtime/current`是到`/srv/runtime/v1`的符号链接。

运行时配置键中的每一个‘.’ 都代表层次结构中的一个新的目录, 扎根于*symlink_root* +*subdirectory*。例如, 键*health_check.min_interval*将具有以下完整文件系统路径（使用符号链接）：

`/srv/runtime/current/envoy/health_check/min_interval`

路径的末端的部分是文件。文件的内容构成运行时值。从文件读取数值时，空格和新行将被忽略。

*override_subdirectory* 与 [`--service-cluster`](../operations/cli.md#cmdoption-service-cluster) 在命令行界面操作时一起使用的。假如 [`--service-cluster`](../operations/cli.md#cmdoption-service-cluster) 被设置成为了`my-cluster`。Envoy将首先从下面完整的文件系统路径中找 *health_check.min_interval*项：

`/srv/runtime/current/envoy_override/my-cluster/health_check/min_interval`

如果找到了,该值将覆盖在主查找路径中找到的任何值。这允许用户在全局默认值之上自定义单个群集的运行时值。

## 注释

以`#`作为第一个字符的行是注释。

注释可以提供现有值的上下文。注释在其他空文件中也很有用，其可以在需要的时候保留占位符以进行部署。

## 通过符号链接交换更新运行时值

更新运行时值总共有两步。第一步, 创建整个运行时树的硬拷贝并更新所需的运行时值。第二步, 使用以下命令的等价物，将旧树中的符号链接根自动交换到新的运行时树：

```
/srv/runtime:~$ ln -s /srv/runtime/v2 new && mv -Tf new current
```

关于如何部署文件系统数据，如何收集垃圾等操作超出了本文的范围。

## 统计

文件系统运行时提供程序在*运行时*发出一些统计信息。命名空间。

| 名称                    | 类型    | 描述                                                  |
| ----------------------- | ------- | ------------------------------------------------------------ |
| load_error              | Counter | 错误重新尝试加载的总数  |
| override_dir_not_exists | Counter | 未使用覆盖目录的加载总数 |
| override_dir_exists     | Counter | 使用覆盖目录的加载总数    |
| load_success            | Counter | 成功加载的尝试总数           |
| num_keys                | Gauge   | 当前加载的键数                              |
