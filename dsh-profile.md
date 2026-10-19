# 测评报告 · @hyzyn/dsh-profile 0.2.9 · Windows 真实宿主增量复核

本件为本地交付，不是已提交/受理的 intake。

## 版本、环境与方法

- 上游：`hyzyn/dsh-plugin-kit`，锚定 **7363332f595b8112cb4b601f8723e6453043eb62**，commit 日期 2026-10-18T13:46:13Z；本轮取证 2026-10-19。
- 宿主 **DSH 0.2.0-rc.2**，Node **v24.19.0**，Playwright **1.64.1**，Python **3.14.7**；Windows-11-10.0.26200-SP0。只测 Windows，未跑 WSL/Linux/macOS。
- 正版 npm 发布 tgz，通过 registry SHA1 / SHA512-integrity，且每包 9 个发布文件与固定 Git 树一致；package.json 按 JSON 语义比对，其余逐字节比对。详见 `INPUTS.json`。
- 复用既有 DSH、Chrome、Playwright、monorepo 依赖和包管理器缓存；不是跨机器从零安装验证。包管理器报告 peer warnings，未隐去。
- 每个插件独立 DSH_HOME / profile / TEMP / 浏览器 context。使用完整 DSH CLI 启动、真实 Chrome 操作实际页面，不以假 ctx、假 webserver 或假凭据 seam 冒充真实宿主。
- 无插件基线真实对应 API 返回 **404 `not found`**；安装后真实 API 返回 **200**。基线是功能可达性对照，不是耗时或模型能力对照。
- 每次命令保留 command/cwd/UTC/exit/stdout/stderr。原目录与新目录各实跑一轮；重跑不是新用例或新缺陷。不改目标源码，不发 PR，不调用付费模型。
- 主目录证据在 `evidence/<插件>/`；新目录复跑证据在 `clean-evidence/<插件>/`；可执行脚本在同名插件目录。原始私有记录另存本机，公开候选包仅含白名单脱敏副本。

## 依据与查重范围

- 工作流规范：[插件测评_AI编程工作流规范_v1.0](https://github.com/FuRongJun-1999/dsh-bounty/blob/main/docs/bounty/插件测评_AI编程工作流规范_v1.0.md)，读取 blob `15658d890c80e559c4f41918171945ffe27dd874`。
- 测评站：[review.lingshulab.com](https://review.lingshulab.com/)，本轮保存首页 HTTP 200；不由首页或“暂无”推断无人评测。
- 在 dsh-bounty 按包名查到 env 59、mcp 22、profile 141 条搜索结果；profile 完整翻两页。数字包含正文/评论噪声，不是相应插件的报告数。已读重点正文/评论与目标仓 issue；查询摘要在 `research/`。
- 已有报告存在，本件为 **Windows 真实 UI / 实际宿主增量复核**，不主张首测。没有新 bug 的插件也只报复核，不借用他人已报漏洞。

## 发布件

`@hyzyn/dsh-profile@0.2.9`

SHA256：`71102fc1c9bdabc58c548b1f8e884a94db8127b3d52e5adb33304004c8b5bddd`

## 上游检查（不替代真实 UI）

- `node node_modules/vitest/vitest.mjs run packages/profile --reporter=verbose`：Tests  3 passed (3)，exit 0。
- 包目录 `node ../../node_modules/typescript/bin/tsc --noEmit`：exit 0。
- 包目录 `node ../../scripts/client-lint.mjs`：exit 0；其自身忽略 TS2339 已知噪音（env 1、mcp 12、profile 9 条），不是完整无告警证明。

## 结果

| 路径 | 原目录 | 新目录复跑 |
|---|---|---|
| 真实浏览器与文件状态 | 34 通过 / 0 失败（34 断言） | 34 通过 / 0 失败（34 断言） |
| 冷重启列表与删除持久化 | 4 通过 / 0 失败（4 断言） | 4 通过 / 0 失败（4 断言） |

**本轮正常路径未发现新的确定产品缺陷。** 不意味着已知 P0 已消除。

## 实际用户路径

- 非法路径名 `../outside`、端口65536从 UI 提交→400；测试根外目标未创建、非法端口无目录半成品。
- 两个独立合成 profile：基础模板、headless模板各建一个，端口0按原值落盘；核对实际 package.json 名字和 bundles，不只看 toast。
- UI prompt将端口改8432→复制启动命令→读取真实浏览器剪贴板，匹配 `dsh --profile <name> --port 8432`。
- 重命名→旧目录消失、新目录 manifest.name 同步；清除端口→列表 port=null。
- 仅从本轮新建、dependencies为空的基础 profile复制到**核验过不存在的新名字**。实际 pnpm install完成，目标 manifest正确；不据此推断任意有依赖的profile可正常复制。
- 删除先取消→副本保留；再确认→只副本消失、源仍在。重启后两个 renamed profile仍已初始化，副本仍不存在。
- 原目录34/0与新目录34/0复跑用例相同，不能计成68个不同功能点。尚未启动新建profile去验证完整功能，仅验文件/列表/复制安装及命令文本。

## 已知高风险和避免误报

bounty **#1007** / 目标仓 **#13** 已报告：复制目标名字已存在或源不存在时，catch可能递归删掉已有目标目录；默认web保护也可被绕过。本轮不在真实 profile触发，也未重复申报/计分。测试全部限定新建合成项，未修改日常profile，更未宣称正常路径全绿等于插件整体无风险。

## 查重与增量

对照 bounty **#1028 / #1007** 和目标仓 issue。#1028 明确真实浏览器与真实安装未测，本件补真实 UI、文件状态和冷重启。profile管理面不直接提供模型工具，本件不套“模型能力五状态”排名。

## 诚实边界与人工事项

- **已验**：本文逐项有读数的层级；浏览器 pageerror 列表为空。新目录复跑使用相同机器/依赖缓存，不等于跨平台或独立第三方验证。
- **未验**：Linux/macOS、生产日常 profile、真实模型/agent 回合、模型效果与能力五状态、跨进程竞态和长期压力。无真实模型请求，不声称“模型一定会正确使用”。
- **不确定**：Windows 文件 ACL/其它宿主版本的兼容性未据此推断。全局配置快照在不同命令之间曾出现 headless profile 新增，来源未归因；各条记录器包围的执行区间内，列出的日常配置哈希均未变，不能扩大成整机从未被其它进程改动。
- **不做**：不扫描网站，不碰真实凭据，不触发已知破坏场景，不排名/星级/总分，不领取或保证贡献分成/报销。
- **人工**：本轮本机测试无需用户重复点击。跨平台、真实模型调用、维护者对契约/严重级别的裁定是后续范围，不冒充已完成。
- **提交状态**：本地报告/证据已整理，**本轮未新建远端仓库、Release、issue 或 PR**。本轮请求为“再测 3 个”；公开投稿另行确认。
