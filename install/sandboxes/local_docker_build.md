# 创建一个 Envoy Docker 镜像

下面的步骤将指导您构建自己的 Envoy 二进制文件，并将其放入一个干净的 Ubuntu 容器中。

**第一步：构建 Envoy**

使用 `envoyproxy/envoy-build` 你可以编译 Envoy。这个镜像包含你构建 Envoy 所有所需的软件。在你的 Envoy 文件中：

```
$ pwd
src/envoy
$ ./ci/run_envoy_docker.sh './ci/do_ci.sh bazel.release'
```

这个命令需要一些时间才能运行，因为它正在编译一个 Envoy 二进制文件并运行测试。
有关构建和不同构建目标的更多信息，请参阅 `repose:ci/README.md`。

**第二步：只使用 envoy 的二进制文件构建镜像**

在这一步中，我们将构建一个只有 Envoy  二进制的镜像，而没有一个软件用于构建它：

```
$ pwd
src/envoy/
$ docker build -f ci/Dockerfile-envoy-image -t envoy .
```

现在如果您在任何Dockerfile中更改了 `FROM`  行，您可以使用这个`envoy`镜像构建任何沙箱。
如果您对修 Envoy 使感兴趣，并测试您的更改，这将特别有用。