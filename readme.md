# sealdice-Sharkbuild-private

本仓库是 Sharkbuild 私有构建仓库，基于 `sealdice/sealdice-build` 的组织方式维护，用来组装并自动构建 Shark 版海豹核心、Web UI 和内置资源。

核心源码不在本仓库直接展开维护，而是通过 `sealdice-core` submodule 固定到私有仓库。

Core: [GuraQwQ/sealdice-Sharkcore-private](https://github.com/GuraQwQ/sealdice-Sharkcore-private)

Build: [GuraQwQ/sealdice-Sharkbuild-private](https://github.com/GuraQwQ/sealdice-Sharkbuild-private)

## 仓库内容

`sealdice-core`

指向 Sharkcore 私有仓库的 git submodule。构建时使用当前提交指针，避免 build 仓库手动展开核心源码。

`sealdice-ui`

Web UI 源码或构建所需内容。

`sealdice-builtins`

内置资源，包含牌堆、helpdoc、图片、名字库等数据。

`scripts`

Docker 和构建辅助脚本。

`.github/workflows`

自动构建、发布、容器镜像推送流程。

首次克隆后执行：

```bash
git submodule update --init --recursive
```

GitHub Actions 读取私有 `sealdice-core` submodule 需要仓库 Secret。

`CORE_REPO_TOKEN`

具备读取 `GuraQwQ/sealdice-Sharkcore-private` 权限的 token。

## 上游同步范围

当前 Sharkcore 已按 `sealdice/sealdice-core` 的 `v1.5.1` 基线继续同步到 upstream `master`。

基线版本：`v1.5.1`

基线日期：`2025-10-10`

已核对范围：`v1.5.1..upstream/master`

已合入 PR 数：123

当前 upstream/master：`e31f443 chore(deps): bump golang.org/x/time from 0.14.0 to 0.15.0 (#1661)`

最新已合并 PR 样例：`#1698`、`#1697`、`#1696`、`#1695`、`#1694`

Sharkcore 在同步上游后保留 Shark 私有改动；这些改动不应被上游同步覆盖。

## Sharkcore 新增功能

### 1. 群文件上传事件 Hook

Sharkcore 新增 OneBot/go-cq 侧群文件上传事件处理，让扩展可以收到 `group_upload` 通知。

对应文件：

`dice/dice.go`

`dice/im_session.go`

`dice/platform_adapter_gocq.go`

`dice/ext.go`

具体改动：

`dice/dice.go`

新增 `GroupUploadFile`，描述 OneBot 11 `group_upload` 事件里的文件信息。

在 `ExtInfo` 中新增 `OnGroupUpload func(ctx *MsgContext, msg *Message, file *GroupUploadFile)` 扩展回调。

`dice/im_session.go`

新增 `IMSession.OnGroupUpload`。

收到群文件上传事件后，查找群信息，补齐 `ctx.Group` / `ctx.Player`，遍历当前群已激活扩展并触发回调。

保留 `DefaultSetting.AutoActive` 兼容逻辑，避免新扩展默认启用设置失效。

`dice/platform_adapter_gocq.go`

在 `MessageQQBase` 中增加 `File` 字段，兼容 go-cq/OneBot 文件上传通知载荷。

新增 `tryHandleGroupUpload`，识别多种 `group_upload` / `notice.group_upload` 结构。

支持从 `file.name`、`file.filename`、`file.file_name` 读取文件名。

在普通消息解析前拦截并派发群文件上传事件，避免被当作普通消息错误处理。

`dice/ext.go`

新增 `ExtInfo.CallOnGroupUpload`，保持 JS 扩展 wrapper 调用路径一致。

`ExtActivateBatch` / `SyncExtensionsOnMessage` 同步兼容 `DefaultSetting.AutoActive`。

### 2. LLBot At 消息段修复

Sharkcore 已补入 upstream commit `2f1ecadd51d47f845edacdb50f84016737c971d5` 同类修复，解决 LLBot/OneBot 发送 at 段时 array message 中没有实际 `at` segment 的问题。

对应文件：

`dice/platform_adapter_onebot_util.go`

`dice/imsdk/onebot/emitter.go`

`dice/platform_adapter_onebot_util_test.go`

具体改动：

`dice/platform_adapter_onebot_util.go`

`convertSealMsgToMessageChain` 中将 `rawMsg.At(res.Target)` 改为 `rawMsg = rawMsg.At(res.Target)`。

原因：`schema.MessageChain.At` 返回追加后的新链；不赋回会导致 CQ 字符串里有 `[CQ:at]`，但 OneBot array message 里没有 `{"type":"at"}`。

`dice/imsdk/onebot/emitter.go`

OneBot action 返回 `failed` 时，在错误信息里附带 raw response，方便排查 LLBot/协议端拒绝原因。

`dice/platform_adapter_onebot_util_test.go`

新增 at 段转换测试，断言 CQ 字符串和 array message 都包含目标 QQ。

验收现象：

```json
{"type":"at","data":{"qq":"2930699167"}}
```

含 at 的分段消息发送时，OneBot array message 中必须出现上述 `at` segment，不再只生成 CQ 字符串。

## 构建与本地工具链

本地仓库自带 Go 工具链：

```powershell
.\.devtools\go1.25.0\bin\go.exe version
.\.devtools\go1.25.0\bin\gofmt.exe -w <files>
```

针对 LLBot at 修复的最小测试：

```powershell
.\.devtools\go1.25.0\bin\go.exe test ./dice -run TestConvertSealMsgToMessageChain_AtElement -count=1
```

更完整的核心检查：

```powershell
.\.devtools\go1.25.0\bin\go.exe test ./dice/imsdk/onebot ./dice -count=1
```

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

注意：

不要在 build 仓库直接展开修改 `sealdice-core` 源码。

上游同步时必须保留 Sharkcore 中 `dice.go`、`im_session.go`、`platform_adapter_gocq.go` 的群文件上传相关新增内容。

LLBot at 段修复属于 Sharkcore 必保补丁，后续上游同步时也不能回退。
