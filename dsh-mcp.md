# 测评报告 · @hyzyn/dsh-mcp 0.3.6 · Windows 真实宿主增量复核

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

`@hyzyn/dsh-mcp@0.3.6`

SHA256：`8289bd13317367deb85eaa734ad9fe535af57b9e6e153ece781abc064d34aca9`

## 上游检查（不替代真实 UI）

- `node node_modules/vitest/vitest.mjs run packages/mcp --reporter=verbose`：Tests  49 passed (49)，exit 0。
- 包目录 `node ../../node_modules/typescript/bin/tsc --noEmit`：exit 0。
- 包目录 `node ../../scripts/client-lint.mjs`：exit 0；其自身忽略 TS2339 已知噪音（env 1、mcp 12、profile 9 条），不是完整无告警证明。

## 结果

| 路径 | 原目录 | 新目录复跑 |
|---|---|---|
| 真实 UI、stdio 与 HMR | 23 通过 / 0 失败（23 断言） | 23 通过 / 0 失败（23 断言） |
| 冷重启 UI/真实注册表计数 | 3 通过 / 0 失败（3 断言） | 3 通过 / 0 失败（3 断言） |

**本轮未发现新的确定产品缺陷**；本件不是旧缺陷的修复验证。

## 实际用户路径

- UI 新建 stdio server，命令为真实 Node 绝对路径，对端原字节复用上游 `scripts/fixtures/mcp-probe-server.mjs`（Apache-2.0，包内 fixtures/LICENSE）。这是合成测试对端，不冒充生产 MCP 服务。
- 编辑器点“连接测试”→ HTTP 200、`result.ok:true`，工具名 **probe_add / probe_echo**。测试后 name/args 未丢失。
- 点保存 → 真 DSH loader/HMR 完成后，`GET /servers` 的活工具注册表统计 **toolCount=2**，不是仅看 fiber active。
- UI 停用→ toolCount **0**、disabled=true；启用→工具数回 **2**。形成 **2→0→2** 的实际运行时对照。
- 编辑成不存在的 `.exe` 做连接测试→ `result.ok:false`、ENOENT；宿主仍 200 可读。此例不等价于上游已报的启动即退出/EPIPE 竞态，**没有证明 #1007 已修**。
- 未带 clearAll 的整表空保存→400；UI 明确确认删除最后一行→200、列表空。等 HMR 真卸载再刷新后才重建，重建工具数2。
- 冷重启工具数仍2。对端 JSONL 有 initialize / tools/list；**未发生真实 agent/model/tools/call，不声称已跑工具业务调用或模型能力五状态。**

## 首次驱动失败（保留，不算产品 bug）

`evidence/mcp/review/RESULT.json`：18 pass / 1 fail，唯一失败是 `driver-completion`。删除响应到达时，旧 fiber 尚未完成异步卸载，返回 DTO 把旧实例暂列 externalServers；驱动马上以同名新增，前端校验拦截，无 save 请求，waitForResponse 超时。

原脚本留为 `mcp/browser-test.r1.mjs`。最终脚本等待 externalServers 中旧名消失，并刷新客户端后再添加；不是延长超时掩盖错误。`review-r2` 23/0，随后新目录最终脚本同样23/0。不要把 18+23 加成41个不同测试。

## 下载/安装失败边界

按包名 npm pack 两次、官方 tarball npm pack、urllib、一次 curl 和 offline cache 均有失败原始记录（ECONNRESET / timeout / ENOTCACHED），未当作插件漏洞。最后由 DNS 查询所得的官方 CDN A 记录 `104.16.0.34` 配合 curl --resolve 取得 **63263 bytes**；URL、TLS/SNI 仍为 registry.npmjs.org，没有关证书校验或换镜像。取件后 SHA1/SHA512 与官方 metadata 对账，再与 Git逐文件对账才装载。此 IP 只描述本轮历史，复跑不要求固定它。

## 查重与增量

对照 bounty **#1023 / #1007 / #975** 和目标仓 issue。#1023 明确未验真实浏览器/完整宿主HMR/真实工具加载，本件补这一层。#1007 的 EPIPE、托管区块子串破坏等不重报；原24+24等探针不搬作本人成绩。没有对 streamable-http、Windows `.cmd` shim、重连策略或并发写下质量结论。

## 诚实边界与人工事项

- **已验**：本文逐项有读数的层级；浏览器 pageerror 列表为空。新目录复跑使用相同机器/依赖缓存，不等于跨平台或独立第三方验证。
- **未验**：Linux/macOS、生产日常 profile、真实模型/agent 回合、模型效果与能力五状态、跨进程竞态和长期压力。无真实模型请求，不声称“模型一定会正确使用”。
- **不确定**：Windows 文件 ACL/其它宿主版本的兼容性未据此推断。全局配置快照在不同命令之间曾出现 headless profile 新增，来源未归因；各条记录器包围的执行区间内，列出的日常配置哈希均未变，不能扩大成整机从未被其它进程改动。
- **不做**：不扫描网站，不碰真实凭据，不触发已知破坏场景，不排名/星级/总分，不领取或保证贡献分成/报销。
- **人工**：本轮本机测试无需用户重复点击。跨平台、真实模型调用、维护者对契约/严重级别的裁定是后续范围，不冒充已完成。
- **提交状态**：本地报告/证据已整理，**本轮未新建远端仓库、Release、issue 或 PR**。本轮请求为“再测 3 个”；公开投稿另行确认。
