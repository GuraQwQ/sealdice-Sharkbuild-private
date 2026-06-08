# sealdice-Sharkbuild-private

本仓库是 Sharkbuild 私有构建仓库，基于 `sealdice/sealdice-build` 的组织方式维护，用来组装并自动构建鲨鲨版海豹核心、Web UI 和内置资源。

核心源码通过 `sealdice-core` submodule 固定到私有仓库。

Core: [GuraQwQ/sealdice-Sharkcore-private](https://github.com/GuraQwQ/sealdice-Sharkcore-private)

Build: [GuraQwQ/sealdice-Sharkbuild-private](https://github.com/GuraQwQ/sealdice-Sharkbuild-private)

## 仓库内容

`sealdice-core`

指向私有仓库的 git submodule。构建时使用当前提交指针，避免 build 仓库手动展开核心源码。

`sealdice-sharkui`

Web UI 源码或构建所需内容。

`sealdice-builtins`

内置资源，包含牌堆、helpdoc、图片、名字库等数据。

`scripts`

Docker 和构建辅助脚本。

`.github/workflows`

自动构建、发布、容器镜像推送流程。

## 同步流程

同步 Sharkcore 后，build 仓库只提交 submodule 指针：

```bash
cd sealdice-core
git fetch origin
git checkout <target-core-commit>
cd ..
git add sealdice-core
git commit -m "chore: update sharkcore submodule"
```
