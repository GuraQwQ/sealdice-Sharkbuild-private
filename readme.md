# sealdice-Sharkbuild-private

本仓库基于 [sealdice/sealdice-build](https://github.com/sealdice/sealdice-build) 结构，用于私有自动构建。

核心仓库：[sealdice-Sharkcore-private](https://github.com/GuraQwQ/sealdice-Sharkcore-private)。

## 仓库内容

- `sealdice-core`：链接到 [`GuraQwQ/sealdice-Sharkcore-private`](https://github.com/GuraQwQ/sealdice-Sharkcore-private) 的 git submodule，build 仓库不再手动展开维护核心源码。
- `sealdice-ui`：来自 `sealdice/sealdice-ui` 的完整前端源码。
- `sealdice-builtins`：来自 `sealdice/sealdice-builtins` 的完整内置资源，包含 `data` 下牌堆、helpdoc、图片、名字库等。
- `scripts`：Docker 打包脚本。

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

已移除：

- `sealdice-android` 源码。
- Android core 构建。
- Android APK 打包、签名、上传。

## 私有核心仓库

`sealdice-core` 是私有仓库 submodule。首次克隆或更新时使用：

```bash
git submodule update --init --recursive
```

GitHub Actions 需要仓库 Secret：

- `CORE_REPO_TOKEN`：有权限读取 `GuraQwQ/sealdice-Sharkcore-private` 的 token，用于 checkout 私有 submodule。

## 同步说明

核心源码只在 [`GuraQwQ/sealdice-Sharkcore-private`](https://github.com/GuraQwQ/sealdice-Sharkcore-private) 中维护。build 仓库同步核心时只需要更新 `sealdice-core` submodule 指针并提交：

```bash
git submodule update --remote sealdice-core
git add sealdice-core
git commit -m "Update core submodule"
```

`sealdice-ui` 与 `sealdice-builtins` 仍按展开目录维护。