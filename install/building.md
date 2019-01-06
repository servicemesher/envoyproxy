# 构建

Envoy 构建系统使用了 Bazel。为了简化初始构建和快速开始，我们提供了一个基于 Ubuntu 16的 docker 容器，其内部包含了可以用来构建和静态链接 envoy 的所有东西，详见 [ci/README.md](https://github.com/envoyproxy/envoy/blob/master/ci/README.md)

手动构建请参考 [bazel/README.md](https://github.com/envoyproxy/envoy/blob/master/bazel/README.md) 的说明。

## 要求

Envoy 最初是在 Ubuntu 14 LTS 上开发和部署的。它应该适用于任何最新的 Linux，包括 Ubuntu 16 LTS。

构建 Envoy 有以下要求:

- GCC 5+ (对 C++14 支持)。
- 这些[预构建](https://github.com/envoyproxy/envoy/blob/master//ci/build_container/build_recipes)的第三方依赖。
- 这些 [Bazel 原生](https://github.com/envoyproxy/envoy/blob/master/bazel/repository_locations.bzl) 的依赖。

请阅读 [CI](https://github.com/envoyproxy/envoy/blob/master/ci/README.md) 和 [Bazel](https://github.com/envoyproxy/envoy/blob/master/bazel/README.md) 的文档。获取有关执行手动构建的更多信息。

## 预构建的二进制文件

在每一个 master 提交中，我们创建了一组轻量级 Docker 镜像，其中包含 Envoy 的二进制文件。我们在发布官方版本时也会使用发布版本号来标记 docker 镜像。

- [envoyproxy/envoy](https://hub.docker.com/r/envoyproxy/envoy/tags/)：发布在 Ubuntu Xenial 基础之上带有标记的二进制文件
- [envoyproxy/envoy-alpine](https://hub.docker.com/r/envoyproxy/envoy-alpine/tags/)：发布在 glibc 基础之上带有标记的二进制文件
- [envoyproxy/envoy-alpine-debug](https://hub.docker.com/r/envoyproxy/envoy-alpine-debug/tags/)：发布在 glibc 基础之上带有 debug 标记的二进制文件

如果可以获得社区在 CI 和打包等方面帮助的话，我们还打算创建额外的二进制类型，如果需要的话，请在 GitHub 上打开一个 [issue](https://github.com/envoyproxy/envoy/issues)。

## 修改 Envoy

如果你对修改 Envoy  和测试变更感兴趣，一种方法就是使用 Docker。本指南将介绍构建自己的 Envoy 二进制文件的过程，并将二进制文件放入一个 Ubuntu 容器中。

- [构建 Envoy Docker 镜像](https://github.com/servicemesher/envoy/blob/master/install/sandboxes/local_docker_build.md)