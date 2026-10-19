# 测评报告 · @hyzyn/dsh-env 0.3.10 · Windows 真实宿主增量复核

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

`@hyzyn/dsh-env@0.3.10`

SHA256：`9a36c3f9a4aaa1e0de40a0de227159c97423b20a8251f6f4d585a7e7c7c5cb3c`

## 上游检查（不替代真实 UI）

- `node node_modules/vitest/vitest.mjs run packages/env --reporter=verbose`：Tests  32 passed (32)，exit 0。
- 包目录 `node ../../node_modules/typescript/bin/tsc --noEmit`：exit 0。
- 包目录 `node ../../scripts/client-lint.mjs`：exit 0；其自身忽略 TS2339 已知噪音（env 1、mcp 12、profile 9 条），不是完整无告警证明。

## 结果

| 路径 | 原目录 | 新目录复跑 |
|---|---|---|
| 无观察夹具的浏览器 UI | 18 通过 / 0 失败（18 断言） | 18 通过 / 0 失败（18 断言） |
| 真实宿主进程环境（只读 observer） | 12 通过 / 2 失败（14 断言） | 12 通过 / 2 失败（14 断言） |
| 重启后真实进程环境 | 6 通过 / 0 失败（6 断言） | 6 通过 / 0 失败（6 断言） |
| 重启后 UI/API | 4 通过 / 0 失败（4 断言） | 4 通过 / 0 失败（4 断言） |
| 额外 2 秒延迟与空值对照（仅原目录） | 16 通过 / 2 失败（18 断言） | 未额外复跑 |

UI 已验：新增普通变量/合成密钥、refs 真实凭据落盘、API `value:null` 与 password 空白、不下发密钥明文、留空保存保留、secret→普通还原值、重新设 secret、删除条目、非法键返回 400 且旧文件逐字节保留。真实 `.credentials.yaml` 使用合成值；不是加密保证，也没有公开该文件。

## 一项新观察：删除行保存未撤销当前 process.env 值（建议 P2 候选）

### 可复跑读数

1. 在 UI 新增 `REVIEW_RUNTIME_A=runtime-A`，点保存 → HTTP 200、`applied:true`；只读观察返回进程值 `runtime-A`。
2. 在 UI 删除该行，再点保存 → HTTP 200、`applied:true`；`GET /api/dsh-env/list` 已无该键，但宿主实际 `process.env.REVIEW_RUNTIME_A` **仍是 `runtime-A`**。
3. B 键重复同一过程，仍保留 `runtime-B`。新 DSH_HOME / 新 profile 复跑同样两条失败；`runtime/RESULT.json` 的失败只限 `runtime-delete-unsets-process-A/B`，无 driver-completion / timeout。
4. 冷重启后，A/B 均为 null；合成普通值和凭据密钥仍正确。残留局限于正在运行的宿主，不是配置未删除或凭据读错。
5. 额外对照将值改成 `delayed-A/B`：删除后等待 **2000ms** 再读仍保留；随后重新加同名行、值显式为空并保存，进程值变为 null。该补充 `delayed-control` 为 **16 通过/2 失败，exit 1**，两红只限删除后未清除。

命令（在全新准备的 env 测试目录，宿主和 observer 已按 REPRODUCE 启动）：

```powershell
python -X utf8 record.py runtime-review . node process-test.mjs runtime
# 预期复现：exit 1，12 pass / 2 fail（仅 runtime-delete-unsets-process-A/B）
python -X utf8 record.py delayed-control . node delayed-control.mjs
# 额外对照：exit 1，16 pass / 2 fail（仅 deleted-after-2s-unset-A/B）
```

### 第一性原理定位

- 保存路由先读 `current.entries`，再校验得到**新列表**，写文件后只把新列表传给 `applyToProcessEnv(entries)`（`packages/env/src/index.ts:590–620`）。
- `applyToProcessEnv`（`:311–324`）只遍历新列表，只有仍存在的条目且求值为 `''` 时才 `delete process.env[key]`。被 UI 删除的键根本不在循环里，没有旧/新键差集撤销步骤。
- 观察夹具不执行任何 set/delete，不枚举环境，仅固定读 5 个 REVIEW 合成键。目标插件、真实宿主、真实凭据 seam 均未替换；详见 `env/observer/index.js`。

### 影响与结论限度

UI 配置视图和实际进程状态不一致；用户以为删掉的普通变量在当前进程仍可见，后续子进程按 Node 默认继承规则可能继续取得旧值（**子进程继承本轮未实测**）。不能据此声称真实密钥泄漏、跨用户攻击或安全撤销被绕过。

**行为高置信，契约定级待确认**：README 承诺“保存即写入当前进程”，但没有明确删除行是否也应立即撤销、是否需还原启动继承值。建议作为 P2 体验/状态一致性候选交维护者确认；若作者明确约定只改持久配置，则改为明确提示“删除需重启”，不硬报安全漏洞。未提交补丁，不声称已修复。

## 查重与增量

对照 bounty **#983 / #975 / #1007**、目标仓 **#13 / #11**，另搜 `dsh-env+删除`、`process.env+残留`、目标仓 `env+删除`。#983 已报 `js:null/undefined/空串` 删除语义和表达式时序；本件是 **删除整行后旧进程值仍在**，不是表达式求值为空，也不是末尾换行增长或丢结束标记。已读材料中未见相同读数；不主张全网绝无先报。查询明细见 `research/ENV_RUNTIME_SUMMARY.json`。

本件补了真实 UI、真实 credentials seam、冷重启，以及只读运行时对账；不照搬他人的 133 探针为自己的成绩。

## 诚实边界与人工事项

- **已验**：本文逐项有读数的层级；浏览器 pageerror 列表为空。新目录复跑使用相同机器/依赖缓存，不等于跨平台或独立第三方验证。
- **未验**：Linux/macOS、生产日常 profile、真实模型/agent 回合、模型效果与能力五状态、跨进程竞态和长期压力。无真实模型请求，不声称“模型一定会正确使用”。
- **不确定**：Windows 文件 ACL/其它宿主版本的兼容性未据此推断。全局配置快照在不同命令之间曾出现 headless profile 新增，来源未归因；各条记录器包围的执行区间内，列出的日常配置哈希均未变，不能扩大成整机从未被其它进程改动。
- **不做**：不扫描网站，不碰真实凭据，不触发已知破坏场景，不排名/星级/总分，不领取或保证贡献分成/报销。
- **人工**：本轮本机测试无需用户重复点击。跨平台、真实模型调用、维护者对契约/严重级别的裁定是后续范围，不冒充已完成。
- **提交状态**：本地报告/证据已整理，**本轮未新建远端仓库、Release、issue 或 PR**。本轮请求为“再测 3 个”；公开投稿另行确认。
