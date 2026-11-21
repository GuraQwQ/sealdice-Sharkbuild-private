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
git commit -m "更新鲨鲨核心构建指针"
```

Core 和 UI 都按本仓库的 gitlink 提交构建。核心版本元数据使用 Core 提交，安装包名称使用 Build 提交；包内的 `build-info.json` 记录 Build、Core、UI 三个完整提交供核对。

当前工具链为 Go 1.25.14、Node.js 22 和 pnpm 11，前端按锁文件安装。Go 版本与官方 Core 要求对齐，Windows 补丁按对应 Go 源码维护；更新工具链时由构建维护者同时验证补丁应用和编译。核心始终嵌入指定 Shark UI 的完整构建产物，生成步骤检查入口资源是否齐全。

`workflow_dispatch` 可以在候选分支生成构建产物，`main` 分支负责发布与镜像推送。同步的行为取舍和迁移说明位于 Core 的 `docs/sync-upstream-20261111.md`。
