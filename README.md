# 三个 DSH 插件 · Windows 真实宿主增量测评

本仓公开用户授权投稿的三份报告及一份共享脱敏证据包；每个插件分别走一个 intake。
**不是首测，不代表已受理。** AI 协助执行和整理，等级/计分由维护者裁定。

| 固定实测对象 | 自带测试 | 真实 Chrome UI | 冷重启 | 结论 |
|---|---|---|---|---|
| [env 0.3.10](dsh-env.md) | 32/32 | 18/18 | UI4/4、进程6/6 | 删行后当前 process.env 残留候选；runtime12通过2失败，契约/建议P2待确认 |
| [MCP 0.3.6](dsh-mcp.md) | 49/49 | 23/23 | 3/3 | 真 stdio 工具注册数2→0→2；独立复核，无新增确定缺陷 |
| [Profile 0.2.9](dsh-profile.md) | 3/3 | 34/34 | 4/4 | 真实新建/复制安装/改名/端口/删除；独立复核，无新增确定缺陷 |

实测提交 `7363332f595b8112cb4b601f8723e6453043eb62`，DSH0.2.0-rc.2。每件新 DSH_HOME/profile 复跑，并由最终 ZIP 新解压后再跑。

## 请先读投稿增补

**[PUBLICATION_ADDENDUM.md](PUBLICATION_ADDENDUM.md)**：投稿前 main 已更新到 `f05c6e8cba2f19609909a7d0866ae37e31d97c07`，
源码声明版本 env 0.3.11 / MCP 0.3.7 / Profile 0.2.10；新版本未运行。
新增提交包含 MCP/Profile 已知缺陷的修复声明，本件不抢报、不声称已验证修复。
报告与 ZIP 原样保留，其中“本轮尚未投稿”等句子是归档前状态，当前实际投稿回执以 intake 及本次发布记录为准。

## 证据下载

- [完整证据 ZIP](https://github.com/says693/dsh-three-plugin-review-20261019/releases/download/review-three-plugins-20261019/THREE_PLUGIN_REVIEW.zip) · [Release](https://github.com/says693/dsh-three-plugin-review-20261019/releases/tag/review-three-plugins-20261019) · [SHA256](THREE_PLUGIN_REVIEW.sha256)
- [复跑说明](REPRODUCE.md) · [最终解压包复跑读数](BUNDLE_REPLAY_VALIDATION.json) · [来源](SOURCES.md)
- ZIP SHA256：`3e5e8c2ba564467da3de45500f67afdf8909d7d3886d85f2a7c732260c3f1189`；405文件，694210 bytes。上传后下载回验，再创建 intake。
- 全量脚本、逐命令退出码/stdout/stderr、逐项断言、脱敏证据和失败尝试在 ZIP 内；不是仅上传汇总。
- 原宿主 token、全量 DSH_HOME、凭据文档、原始 DOM、本机配置与完整 preset stdout 未公开；脚本原字节、证据文本仅路径脱敏。

Windows 单平台、复用本机依赖缓存，不是跨机器全新安装；无 agent/model 调用，不给能力五状态/总分。
运行时合成密钥验证不等于加密承诺；MCP 注册工具不等于 tools/call；Profile 全绿不抹除旧版已知风险。
