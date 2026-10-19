# 复跑与人工交接

这是本地可交付证据包，**尚未投稿**。先读三个报告的已知风险与边界。只在新目录使用，不指向日常 DSH_HOME。本包没有付费模型调用。

## 准备

依赖：Windows、Python3、Node24、Chrome、DSH0.2.0-rc.2、Playwright1.64.1。可复用已安装版本；新装时用独立依赖目录与官方包管理器。本轮实际复用既有安装/缓存，不声称从零装依赖验证。

1. 解压到全新目录；根目录应有 env/、mcp/、profile/、clean-repro.py。
2. 每个插件目录复制 config.example.json 为 config.local.json，填 `dsh_cli`、`playwright_module`、`chrome_exe` 三个真实绝对文件路径（不是目录）。其余 profile/package/version 字段保留；host_tag 由复跑器管理。
3. 核验根 FILES.sha256、INPUTS.json 的目标 tgz SHA256，保留包内 Apache-2.0 LICENSE。不要修改或重打包目标插件；`mcp/fixtures/mcp-probe-server.mjs` 为上游原字节复用。
4. 全自动新目录复跑：

```powershell
python -X utf8 clean-repro.py
```

复跑器在同目录创建新的 clean-repro/，为每包设置新 DSH_HOME/TEMP/profile；实际依次 init→无插件基线404→安装官方tgz→启动真宿主→Chrome UI→停宿主→重启→验证。env 再装只读观察夹具验证真实进程并重启。只停止自己创建的宿主，不停止其它 Node。已经有 clean-repro/ 时拒绝覆盖，换新目录再跑。

预期：env UI18/18、restart4/4、runtime12通过2失败（仅 runtime-delete-unsets-process-A/B）、runtime-restart6/6；mcp23/23、restart3/3；profile34/34、restart4/4。整体复跑器 exit0 表示**得到预先登记的当前版本结果**，不是“所有产品断言通过”；env 内部 runtime 命令明确为 exit1，两红原样保留。

如出现 driver-completion、timeout、额外失败、宿主非零退出或哈希监控失败，保留记录并先排查，**不要称为相同缺陷已复现**。

## 手工逐步跑一件（可替代复跑器）

以下在一个全新 env 测试目录中进行，先按上文填 config.local.json，保留同目录的 record.py/runtime.mjs/test-kit.mjs/host.mjs/browser-test.mjs/published/observer/。

```powershell
$cfg = Get-Content config.local.json -Raw | ConvertFrom-Json
$cli = $cfg.dsh_cli
$name = $cfg.profile
python -X utf8 record.py profile-init . node $cli --profile $name --from-default-profile web --dump-config
# config.local.json 中 host_tag 设 baseline，启动宿主：
python -X utf8 record.py baseline-host . node host.mjs
# 另一个终端同目录执行，随后停止本次宿主：
python -X utf8 record.py baseline-probe . node baseline.mjs
New-Item -ItemType File evidence/baseline.stop
```

安装/真实UI：

```powershell
$tgz = (Resolve-Path published/hyzyn-dsh-env-0.3.10.tgz).Path
python -X utf8 record.py plugin-install . node $cli plugin --profile $name add $tgz
# host_tag 改 installed；不要重用旧日志名
python -X utf8 record.py installed-host . node host.mjs
# 另一终端：
python -X utf8 record.py browser-review . node browser-test.mjs review
New-Item -ItemType File evidence/installed.stop
```

进程环境观察：先停宿主，安装包内已有 observer tgz，再把 host_tag 改 observed。它只返回5个固定合成键，**不是假凭据服务，不修改process.env**。

```powershell
$obs = (Resolve-Path published/dsh-env-review-observer-1.0.0.tgz).Path
python -X utf8 record.py observer-install . node $cli plugin --profile $name add $obs
python -X utf8 record.py observed-host . node host.mjs
# 另一终端：
python -X utf8 record.py runtime-review . node process-test.mjs runtime
# 此处应 exit1 / 12通过2失败；记录不要删。
New-Item -ItemType File evidence/observed.stop
```

host_tag 改 runtime-restart，重启宿主再跑：

```powershell
python -X utf8 record.py runtime-restart-host . node host.mjs
# 另一终端：
python -X utf8 record.py restart-check . node process-test.mjs runtime-restart
# 预期6/6，A/B均null；普通/密钥仍正确。
New-Item -ItemType File evidence/runtime-restart.stop
```

额外延迟/空值对照须在新宿主tag controls 启动（observer保留），执行 `python -X utf8 record.py delayed-control . node delayed-control.mjs`。它等待删除保存后2000ms，再用显式空值对照，预期16通过2失败；最后 `New-Item -ItemType File evidence/controls.stop`。这项主目录实跑过，新目录主流程未额外重复。

MCP/profile 可用同一 init/基线/安装/启动方式，在各自目录运行 browser-test.mjs review 和重启后的 browser-test.mjs restart。MCP对端会被真实宿主启动；不要把它替换成mock。Profile只复制新建基础源到新名字，不做已知危险的同名复制。

## 证据与隐私

- evidence/：原目录命令/断言的脱敏副本；clean-evidence/：新目录复跑副本；各项 pass/fail未改。
- 省略：host-local带令牌URL、process日志、profile-init完整preset stdout、全量DSH_HOME、凭据文件、日常配置内容。省略项未声称公开全部原始字节。
- 脚本原字节复制并验哈希，**不对脚本做路径/token通用替换**；只对证据副本替换本机路径，PROVENANCE逐文件记原始/交付哈希。
- 新建宿主本机URL在 evidence/<tag>-host-local.json，包含会话token，仅本机使用；不要发群、放issue或截地址栏。

## 真需要人工的情况

本轮无强制人工补测。若后续需要：

- **公开投稿**：确认允许公开三份报告、白名单证据以及是否沿用既有证据仓；授权后再刷新查重、按一次一件intake提交。不把“测完”写成“已受理”。
- **Linux/macOS**：用同版本DSH/Node另跑，不拿Windows数据填过去；本驱动含Windows复跑器，跨平台需显式适配并记录。
- **真实模型效果**：先确定模型、次数、预算；用户在本机配置密钥，不把key发给聊天。没有这一步，不给模型能力评级。
- **env契约**：维护者确认“删除行保存”是否承诺立即撤销当前进程值；行为已复现，定级仍待裁定。这不是要求用户重复点同样按钮。
