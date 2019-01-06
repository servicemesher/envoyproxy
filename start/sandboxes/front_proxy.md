# 前端代理

为了帮助大家了解如何使用 Envoy 作为前端代理，我们发布了一个 [docker compose](https://docs.docker.com/compose/) 沙箱，该沙箱中部署了一个前端 envoy 以及与服务 envoy 搭配的一组服务（简单的沙箱应用）。这三个容器将被部署在名为 `envoymesh` 的虚拟网络中。

下面是使用 docker compose 部署的架构图：

![../../images/docker_compose_v0.1.svg](../../images/docker_compose_v0.1.svg)

所有传入的请求都通过前端 envoy 进行路由，该 envoy 充当位于 `envoymesh` 网络边缘的反向代理。通过docker compose 将端口 80 映射到 8000 端口（请参阅 [/examples/front-proxy/docker-compose.yml](https://github.com/envoyproxy/envoy/blob/master//examples/front-proxy/docker-compose.yml)）。此外，请注意，由前端 envoy 由到服务容器的所有流量实际上路由到服务 envoy（在 [/examples/front-proxy/front-envoy.yaml](https://github.com/envoyproxy/envoy/blob/master//examples/front-proxy/front-envoy.yaml) 中设置的路由）。反过来，服务 envoy 通过回环地址（[/examples/front-proxy/service-envoy.yaml](https://github.com/envoyproxy/envoy/blob/master//examples/front-proxy/service-envoy.yaml) 中的路由设置）将请求路由到 flask 应用程序。此设置说明了运行服务 envoy 与您的服务搭配的优势：所有请求都由服务 envoy 处理，并有效地路由到您的服务。

## 运行 Sandbox

以下文档通过按照上图中所述组织的 envoy 集群的设置运行。

**步骤 1：安装 Docker**

确保您已经安装了最新版本的 `docker`、`docker-compose` 和 `docker-machine`。

安装这些软件最简单的方式是使用 [Docker Toolbox](https://www.docker.com/products/docker-toolbox)。

**步骤 2：配置 Docker Machine**

首先创建一个容纳容器的新机器：

```bash
$ docker-machine create --driver virtualbox default
$ eval $(docker-machine env default)
```

**步骤 3：克隆 Envoy repo，启动所有的容器**

如果您还没有克隆 envoy repo，执行 `git clone git@github.com:envoyproxy/envoy` 或者 `git clone https://github.com/envoyproxy/envoy.git` 来克隆。

```bash
$ pwd
envoy/examples/front-proxy
$ docker-compose up --build -d
$ docker-compose ps
        Name                       Command               State      Ports
-------------------------------------------------------------------------------------------------------------
example_service1_1      /bin/sh -c /usr/local/bin/ ... Up       80/tcp
example_service2_1      /bin/sh -c /usr/local/bin/ ... Up       80/tcp
example_front-envoy_1   /bin/sh -c /usr/local/bin/ ... Up       0.0.0.0:8000->80/tcp, 0.0.0.0:8001->8001/tcp
```

**步骤 4：测试 Envoy 的路由能力**

您现在可以通过前端 envoy 向两项服务发送请求。

向 service1：

```bash
$ curl -v $(docker-machine ip default):8000/service/1
*   Trying 192.168.99.100...
* Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
> GET /service/1 HTTP/1.1
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

向 service2：

```bash
$ curl -v $(docker-machine ip default):8000/service/2
*   Trying 192.168.99.100...
* Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
> GET /service/2 HTTP/1.1
> Host: 192.168.99.100:8000
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 89
< x-envoy-upstream-service-time: 2
< server: envoy
< date: Fri, 26 Aug 2016 19:39:23 GMT
< x-envoy-protocol-version: HTTP/1.1
<
Hello from behind Envoy (service 2)! hostname: 92f4a3737bbc resolvedhostname: 172.19.0.2
* Connection #0 to host 192.168.99.100 left intact
```

请注意，每个请求在发送给前端 envoy 时已正确路由到相应的应用程序。

**步骤 5：测试 Envoy 的负载均衡能力**

现在扩展我们的 service1 节点来演示 envoy 的集群能力：

```bash
$ docker-compose scale service1=3
Creating and starting example_service1_2 ... done
Creating and starting example_service1_3 ... done
```

现在，如果我们多次向 service1 发送请求，前端 envoy 将通过循环轮询三台 service1 机器来负载均衡请求：

```bash
$ curl -v $(docker-machine ip default):8000/service/1
*   Trying 192.168.99.100...
* Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
> GET /service/1 HTTP/1.1
> Host: 192.168.99.100:8000
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 89
< x-envoy-upstream-service-time: 1
< server: envoy
< date: Fri, 26 Aug 2016 19:40:21 GMT
< x-envoy-protocol-version: HTTP/1.1
<
Hello from behind Envoy (service 1)! hostname: 85ac151715c6 resolvedhostname: 172.19.0.3
* Connection #0 to host 192.168.99.100 left intact
$ curl -v $(docker-machine ip default):8000/service/1
*   Trying 192.168.99.100...
* Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
> GET /service/1 HTTP/1.1
> Host: 192.168.99.100:8000
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 89
< x-envoy-upstream-service-time: 1
< server: envoy
< date: Fri, 26 Aug 2016 19:40:22 GMT
< x-envoy-protocol-version: HTTP/1.1
<
Hello from behind Envoy (service 1)! hostname: 20da22cfc955 resolvedhostname: 172.19.0.5
* Connection #0 to host 192.168.99.100 left intact
$ curl -v $(docker-machine ip default):8000/service/1
*   Trying 192.168.99.100...
* Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
> GET /service/1 HTTP/1.1
> Host: 192.168.99.100:8000
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 89
< x-envoy-upstream-service-time: 1
< server: envoy
< date: Fri, 26 Aug 2016 19:40:24 GMT
< x-envoy-protocol-version: HTTP/1.1
<
Hello from behind Envoy (service 1)! hostname: f26027f1ce28 resolvedhostname: 172.19.0.6
* Connection #0 to host 192.168.99.100 left intact
```

**步骤 6：进入容器并 curl 服务**

除了使用主机上的 `curl` 外，您还可以进入容器并从容器里面 `curl`。要进入容器，可以使用 `docker-compose exec <容器名> /bin/bash`。例如，我们可以进入前端 envoy 容器，并在本地 `curl` 服务：

```bash
$ docker-compose exec front-envoy /bin/bash
root@81288499f9d7:/# curl localhost:80/service/1
Hello from behind Envoy (service 1)! hostname: 85ac151715c6 resolvedhostname: 172.19.0.3
root@81288499f9d7:/# curl localhost:80/service/1
Hello from behind Envoy (service 1)! hostname: 20da22cfc955 resolvedhostname: 172.19.0.5
root@81288499f9d7:/# curl localhost:80/service/1
Hello from behind Envoy (service 1)! hostname: f26027f1ce28 resolvedhostname: 172.19.0.6
root@81288499f9d7:/# curl localhost:80/service/2
Hello from behind Envoy (service 2)! hostname: 92f4a3737bbc resolvedhostname: 172.19.0.2
```

**步骤7：进入容器并 curl admin**

当 envoy 运行时，它也会将 `admin` 连接到所需的端口。在示例配置 admin 绑定到 8001 端口。我们可以 `curl` 它获得有用的信息。例如，您可以 `curl /server_info` 来获取正在运行的 envoy 版本的信息。此外，你可以 `curl /stats` 来获得统计数据。例如在 `frontenvoy` 里面我们可以得到：

```bash
$ docker-compose exec front-envoy /bin/bash
root@e654c2c83277:/# curl localhost:8001/server_info
envoy 10e00b/RELEASE live 142 142 0
root@e654c2c83277:/# curl localhost:8001/stats
cluster.service1.external.upstream_rq_200: 7
...
cluster.service1.membership_change: 2
cluster.service1.membership_total: 3
...
cluster.service1.upstream_cx_http2_total: 3
...
cluster.service1.upstream_rq_total: 7
...
cluster.service2.external.upstream_rq_200: 2
...
cluster.service2.membership_change: 1
cluster.service2.membership_total: 1
...
cluster.service2.upstream_cx_http2_total: 1
...
cluster.service2.upstream_rq_total: 2
...
```

请注意，我们可以获取上游集群的成员数量，它们实现的请求数量，有关 http 入口的信息以及大量其他有用的统计信息。
