# Zipkin 追踪


Zipkin 追踪 sandbox 使用 [Zipkin](http://zipkin.io/) 作为追踪提供者演示Envoy的[请求追踪](../../intro/arch_overview/tracing.md＃arch-overview-tracing) 功能。此沙箱与上述前端代理体系结构非常相似，但有一点不同：在返回响应之前，service1 会对 service2 进行 API 调用。 这三个容器将被部署在名为`envoymesh`的虚拟网络中。

所有传入的请求都通过前端 envoy 进行路由，该envoy 充当位于 envoymesh 网络边缘的反向代理。 端口`80`通过 docker compose 映射到端口 `8000`（参见[/examples/zipkin-tracing/docker-compose.yml](https://github.com/envoyproxy/envoy/blob/master//examples/zipkin-tracing/docker-compose.yml)）。请注意，所有 envoy 都配置为收集请求跟踪（例如 http_connection_manager/config/tracing 中的设置[/examples/zipkin-tracing/front-envoy-zipkin.yaml](https://github.com/envoyproxy/envoy/blob/master//examples/zipkin-tracing/front-envoy-zipkin.yaml)）并设置为将 Zipkin 追踪器生成的跨度传播到 Zipkin 集群中（跟踪驱动程序设置[/examples/zipkin-tracing/front-envoy-zipkin.yaml](https://github.com/envoyproxy/envoy/blob/master//examples/zipkin-tracing/front-envoy-zipkin.yaml)）。

在将请求路由到相应的服务 envoy 或应用程序之前，Envoy 将负责为跟踪生成适当的跨度（父/子/共享上下文跨度）。 在高层次上，每个跨度记录上游API调用的延迟以及将跨度与其他相关跨度（例如跟踪ID）关联所需的信息。

从 Envoy 进行跟踪的最重要的好处之一是，它将负责将跟踪传播到 Zipkin 服务群集。 但是，为了充分利用跟踪，应用程序必须传播 Envoy 生成的跟踪标头，同时调用其他服务。 在我们提供的沙箱中，简单的应用程序（请参阅 [/examples/front-proxy/service.py](https://github.com/envoyproxy/envoy/blob/master//examples/front-proxy/service.py)）作为 service1 传播跟踪头，同时对 service2 进行出调用。

## 运行 Sandbox

以下文档通过按照上图中所述组织的 envoy 集群的设置运行。


**第1步：构建 sandbox**

要构建这个沙盒示例，并启动示例应用程序，请运行以下命令：

```bash
$ pwd
envoy/examples/zipkin-tracing
$ docker-compose up --build -d
$ docker-compose ps
        Name                       Command               State      Ports
-------------------------------------------------------------------------------------------------------------
zipkintracing_service1_1      /bin/sh -c /usr/local/bin/ ... Up       80/tcp
zipkintracing_service2_1      /bin/sh -c /usr/local/bin/ ... Up       80/tcp
zipkintracing_front-envoy_1   /bin/sh -c /usr/local/bin/ ... Up       0.0.0.0:8000->80/tcp, 0.0.0.0:8001->8001/tcp
```

**第2步：生成一些负载**


您现在可以通过前台特使向 service1 发送请求，如下所示：

```bash
$ curl -v $(docker-machine ip default):8000/trace/1
*   Trying 192.168.99.100...
* Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
> GET /trace/1 HTTP/1.1
> Host: 192.168.99.100:8000
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 89
< x-envoy-upstream-service-time: 1
< server: envoy
< date: Fri, 26 Aug 2016 19:39:19 GMT
< x-envoy-protocol-version: HTTP/1.1
<
Hello from behind Envoy (service 1)! hostname: f26027f1ce28 resolvedhostname: 172.19.0.6
* Connection #0 to host 192.168.99.100 left intact
```

**第3步：在 Zipkin UI 中查看追踪**

使用您的浏览器访问 <http://localhost:9411>。 你应该看到 Zipkin 仪表板。 如果这个 IP 地址不正确，你可以运行：` $ docker-machine ip default `来找到正确的地址。, 将服务设置为“前台代理”，并将开始时间设置为在测试开始前几分钟（步骤2）并按回车。你应该看到前端代理的痕迹。 单击一个跟踪来探索从前端代理到service1到service2的请求所采用的路径，以及每个跃点产生的延迟。
