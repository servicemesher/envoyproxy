# 路由表检查工具

**注意:下面的配置仅用于路由表检查工具，而不是Envoy二进制文件的一部分。路由表检查工具是一个独立的二进制文件，它可以用来验证Envy对给定配置文件的路由。**

下面指定了路由表检查工具的输入。路由表检查工具检查[路由器](../../api-v1/route_config/route_config.md#config-http-conn-man-route-table)返回的路由信息是否符合预期。该工具可用于检查集群名称、虚拟集群名称、虚拟主机名、手动路径重写、手动主机重写、路径重定向和头字段匹配。 可以添加其它测试用例的扩展。安装工具和示例工具输入/输出的详细信息可以在[这里](../../install/tools/route_table_check_tool.md#install-tools-route-table-check-tool)找到。

路由表检查工具配置由一系列json测试对象组成。每个测试对象由三个部分组成。

- Test name

该字段指定每个测试对象的名称。

- Input values

  输入值字段指定要传递给路由器的参数。例如输入字段包括`:authority`，`:path`,`:method header fields`。权限：`path`字段指定发送到路由器的`url`并是必须要添加的。所有其他输入字段都是可选的。

- Validate

  validate字段指定要检查的期望值和测试用例。至少需要一个测试用例。
  
 看下面的一个简单的json工具配置参数，其包含有一个测试用例。测试期望与“instant-server”的集群名称匹配。：

```json
[
  {
    "test_name: "Cluster_name_test",
    "input":
      {
        ":authority":"api.lyft.com",
        ":path": "/api/locations"
      },
    "validate":
      {
        "cluster_name": "instant-server"
      }
   }
]
```

```json
[
  {
    "test_name": "...",
    "input":
      {
        ":authority": "...",
        ":path": "...",
        ":method": "...",
        "internal" : "...",
        "random_value" : "...",
        "ssl" : "...",
        "additional_headers": [
          {
            "field": "...",
            "value": "..."
          },
          {
             "..."
          }
        ]
      },
    "validate": {
      "cluster_name": "...",
      "virtual_cluster_name": "...",
      "virtual_host_name": "...",
      "host_rewrite": "...",
      "path_rewrite": "...",
      "path_redirect": "...",
      "header_fields" : [
        {
          "field": "...",
          "value": "..."
        },
        {
          "..."
        }
      ]
    }
  },
  {
    "..."
  }
]
```

- test_name

  *(required, string)* 测试对象的名称.

- input路径法

  *(required, object)* 输入值发送到路由器，以确定返回的路由信息：`authority` *(required, string)* `url`权限。这个值与`path`参数一起定义要匹配的url。例如权限的值是“`api.lyft.com`”。:`path` *(required, string)* `url`的路径。例如路径的值是 “`/foo`”。`:method` *(optional, string)* 请求的方法。如果没有指定，默认采用`get`方法。这里可以选择`GET`方法, `PUT`方法或者`POST`方法。`internal` *(optional, boolean)* 它是是否将`x-envoy-internal`设置为“true”的一个标志。如果没有指定或者指定为“false”，`x-envoy-internal`就没有配置。`random_value`*(optional, integer)* 用于确定加权集群选择目标的整数。默认值是0。`ssl` *(optional, boolean)* 决定将 `x-forwarded-proto`设置为`http`或者`https`。通过给定的协议设置`x-forwarded-proto`，该工具能够模拟客户机通过`http`或`https`发出请求的行为。 默认情况下`ssl`是`false`，相当于将`x-forwarded-proto`设置为`http`。`additional_headers`*(optional, array)* `Additional headers`将作为路径确定的输入添加。“`:authority`”, “`:path`”, “`:method`”, “`x-forwarded-proto`”以及 “`x-envoy-internal`” 字段应该有其他配置选项指定，而不应该在这里配置。`field` *(required, string)* 要添加的头字段的名称。`value` *(required, string)* 要添加的投字段的值。

- validate

  *(required, object)* `validate`对象指定要匹配的返回路由参数。至少需要指定一个参数， 使用“” (空字符串)来表示没有返回值。 例如, 测试没有集群能够匹配{“`cluster_name`”: “”}。`cluster_name` *(optional, string)* 匹配集群的名称。`virtual_cluster_name` *(optional, string)* 匹配虚拟集群的名称。`virtual_host_name` *(optional, string)* 匹配虚拟机的名称。`host_rewrite` *(optional, string)* 匹配重写后的主机字段头。`path_rewrite` *(optional, string)* 匹配重写后的路径字段头。`path_redirect` *(optional, string)* 匹配返回的重定向路径。`header_fields` *(optional, array)* 匹配列出的头字段。例如头字段包括“`:path`”, “`cookie`”, 以及 “`date`” 字段。在其它测试用例之后检查头字段。因此, 所检查的头字段将是重定向或重写路由信息。`field` *(required, string)* 要匹配的头字段的名称。`value` *(required, string)* 要匹配的头字段的值。
