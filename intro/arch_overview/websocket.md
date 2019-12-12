# WebSocket 支持

Envoy 支持将 HTTP/1.1 连接升级到 WebSocket 连接。仅当下游客户端发送正确的升级请求头，并且被匹配的 HTTP 路由必须明确配置使用了 WebSocket （[use_websocket](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-use-websocket)）时才允许连接升级。如果请求到达 WebSocket 的路由没有必要的升级头，它将被视为常规的 HTTP/1.1 请求。

由于 Envoy 将 WebSocket 连接视为纯 TCP 连接，因此它支持 WebSocket 协议的所有内容，而与它们的报文格式无关。WebSocket 路由不支持某些 HTTP 请求级别的功能，例如重定向、超时、重试、速率限制和阴影。但是，支持前缀重写、显式重写和自动主机重写、流量转移和拆分。

## 连接语义

尽管 WebSocket 升级可以通过 HTTP/1.1 连接进行，但 WebSockets 代理的工作模式与普通 TCP 代理类似，即 Envoy 不会解析 websocket 报文帧。下游客户端和/或上游服务器负责终止 WebSocket 连接（例如，通过发送[关闭帧](https://tools.ietf.org/html/rfc6455#section-5.5.1)）和底层 TCP 连接。

当连接管理器通过支持 WebSocket 的路由接收到 WebSocket 升级请求时，它通过 TCP 连接将请求转发给上游服务器。Envoy 不知道上游服务器是否拒绝了升级请求。上游服务器负责终止 TCP 连接，这会导致 Envoy 终止相应的下游客户端连接。
