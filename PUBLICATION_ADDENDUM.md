# 投稿前增补 · 2026-10-19

## 状态与版本边界

用户在三件测评交付后明确要求投稿。本次按三件独立 intake 发布；是否受理、贡献等级与后续动作由维护者裁定。
原报告和 ZIP 中“尚未投稿”等字样是封存时的历史状态，不是当前投稿状态。原 ZIP 不重打包，不修改原始 pass/fail 或失败记录。

- 固定实测提交：`7363332f595b8112cb4b601f8723e6453043eb62`；env 0.3.10 / MCP 0.3.6 / Profile 0.2.9；宿主 DSH0.2.0-rc.2。
- 投稿前刷新时间：`2026-10-19T14:45:48.879863+00:00`。
- 此时上游 main：`f05c6e8cba2f19609909a7d0866ae37e31d97c07`，比实测提交前进 6 个提交。
- 新 HEAD 源码声明：env 0.3.11 / MCP 0.3.7 / Profile 0.2.10。
- npm dist-tags 快照：`{"env": null, "mcp": null, "profile": null}`。null表示本轮网络请求未确认，成功元数据及失败记录均存私有审计记录；是否发布不由源码版本号推定。
- **没有运行这些新版本**，没有把旧版 UI、单测、重启读数平移给新版。

## 差异阅读

[固定 compare](https://github.com/hyzyn/dsh-plugin-kit/compare/7363332f595b8112cb4b601f8723e6453043eb62...f05c6e8cba2f19609909a7d0866ae37e31d97c07)，134 个变更文件，未达到 compare 的文件截断界限。后续一次推进只改市场索引文档/loglens截图声明，env/MCP/Profile的compare差异与前一份已读快照逐项一致。

1. env src/index.ts 的差异只涉及 registerLiteralSection 导入与公告注册调用；`applyToProcessEnv` 和保存路由本次 compare 未变。只能说明候选根因所在代码未改，**不代替新版本运行复现**。删除行是否应立即撤销进程值、建议 P2 是否合适，待维护者确认。
2. MCP 新提交声明处理 stdin/EPIPE 和整行匹配托管区块，已读相关源码差异。本轮旧版正常 stdio / HMR / ENOENT 读数不证明旧 EPIPE 缺陷修复，也不重复申报旧缺陷。
3. Profile 新提交给 duplicate 回滚增加“本次实际创建目录”跟踪及测试。旧版的 #1007/#13 高风险说明仍仅适用于报告锚定版本；新版本是否实际修复本轮未验，不能把旧警告无条件扩展给新版。

## 刷新查重与计数

- 工作流 blob `15658d890c80e559c4f41918171945ffe27dd874`，与实测时相同。
- dsh-bounty 包名查询 env 59 / MCP 23 / Profile 144，全部页已取齐；它们是文本命中，不是相应插件报告数。
- 本账号 env 0 / MCP 0；Profile 2 个文本命中实际是 #1082 code-security 与 #1089 prompt，不是本轮 Profile 管理面贡献。
- 新命中 #1112 为 appthin/dsh-mcp-manager-plus，不是 hyzyn/dsh-mcp；#1123/#1100/#1095 也分别是其他插件，不作为本轮同范围先报。
- 已再读 #983 / #975 / #1007 / #1023 / #1028 的正文评论及目标仓 #13；env 的整行删除残留未在已读材料中见同读数，MCP/Profile 明确是补真实 UI 与宿主的独立复核。若维护者指出同范围先报，接受合并，不重复计同一贡献。

## 公开与复跑校验

- 405 文件证据 ZIP：SHA256 `3e5e8c2ba564467da3de45500f67afdf8909d7d3886d85f2a7c732260c3f1189`，694210 bytes。
- ZIP CRC、文件清单、解压字节与交付目录一致；包含嵌套 tgz 的敏感字节检查通过。本机真实 host token 未公开。
- 最终 ZIP 已在本机新解压目录复跑，详见 BUNDLE_REPLAY_VALIDATION.json。整体复跑器 exit0 是“得到预登记结果”，env runtime 仍明确 exit1、12通过/2失败，不是全绿。
- 本轮无修复代码、无新版本回归验证，无模型调用、无双平台或跨机器空缓存结论。
