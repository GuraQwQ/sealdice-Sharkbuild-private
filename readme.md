# SealDice Private Build

本仓库基于 [sealdice/sealdice-build](https://github.com/sealdice/sealdice-build) 结构，用于私有自动构建。

## 仓库内容

- `sealdice-core`：来自 `GuraQwQ/sealdice-core-private` 的完整核心源码。
- `sealdice-ui`：来自 `sealdice/sealdice-ui` 的完整前端源码。
- `sealdice-builtins`：来自 `sealdice/sealdice-builtins` 的完整内置资源，包含 `data` 下牌堆、helpdoc、图片、名字库等。
- `scripts`：Docker 打包脚本。

本仓库不使用 git submodule，所有源码和资源均展开提交，GitHub Actions checkout 后即可构建。

## 自动构建

工作流：`.github/workflows/auto-build.yml`

保留：

- `resources-download`：上传 `sealdice-builtins/data` 作为打包资源。
- `ui-build`：构建 Web UI，并注入 `sealdice-core/static/frontend`。
- `core-build`：构建 Linux/Windows core。
- `core-darwin-build`：构建 macOS core。
- `pc-pack`：打包 Windows、Linux、macOS 发行包，包含 core、UI、内置资源、Lagrange/LagrangeV2/Yogurt。
- `preparation` / `prerelease`：压缩并发布预发布产物。
- `docker-push` / `docker-push-full`：构建并推送容器镜像。

移除：

- `sealdice-android` 源码。
- Android core 构建。
- Android APK 打包、签名、上传。
- submodule 自动更新流程。

## 同步说明

同步上游时分别更新 `sealdice-ui`、`sealdice-builtins`、`sealdice-core` 目录内容，再提交到本仓库。注意保留 `sealdice-core/dice/dice.go`、`sealdice-core/dice/im_session.go`、`sealdice-core/dice/platform_adapter_gocq.go` 中本地自定义改动。
