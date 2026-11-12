# config 最新基线受控修复与验证报告

状态：`RESIDUAL_UNCERTAIN`（执行 Owner 自检；等待 PMO 指派独立验收）  
日期：2026-08-09（Asia/Shanghai）  
角色：`2-2 config最新基线受控修复Owner`，唯一执行 Owner  
唯一 live 修改对象：`/Users/USER/.codex/config.toml`

## 1. 结论

已基于开工时最新 live config 完成最小补丁：默认新执行任务改为 `gpt-5.6-terra` + `medium`；模型/Reasonix 路由注释改指全局 AGENTS 第 3 节；用户点名的 4 个旧项目精确块最终计数均为 0；shadcn MCP 从动态 `@latest` 固定为官方 registry 已复核的 `4.16.2`。未改任何 secret/token/env 值、未列出的 MCP、notice 或其他 project 设置。

不能给出绿色完成结论：原审计 L-001 的 4 个路径与本次点名清单不完全一致，仍有 3 个审计路径块未获删除授权；Node REPL 的广域 trusted code path 虽已证明安全语义，但应用自动写回与兼容性使最小安全收紧无法充分证明，因此按用户要求保持原状并列 `UNCERTAIN`。

## 2. 权威输入与开工身份

- 原审计报告 SHA-256：`9e2bd92aa000d1b3f31116ec425e2ad29b24719b3e13e03c146bb0202d28da5f`。
- 主修复一屏回报 SHA-256：`2998712ff3f50a12c1ff7a6524a971f8d5ecfb25c1c6a2ba61e53f09f5c9225a`。
- 主修复映射报告 SHA-256：`ddb34a9bcc26750edea90a8368e3ac0d8808435483261bf65bdd384e94d86183`。
- 主修复 manifest SHA-256：`9b31c3132619727b16a15e631d55e4bc727eb85cbcc19f1280f88d2a2f0d0bfe`。
- 最新 `/Users/USER/.codex/AGENTS.md`：SHA-256 `300388122eb56b0c898de1fa995ccdba32577e1b4f3fa50f21591e7b22c83cee`；mtime `2026-08-09T15:59:40+0800`；inode `40734649`；mode `0644`；owner `cczij`。已完整读取；`AGENTS.override.md` 不存在。
- config 开工身份：SHA-256 `31a30d48b896e9b5be1cc502bbb945a15c15ede296e63d0d786ba247bdf79aad`；mtime `2026-08-09T16:24:25+0800`；inode `55057589`；mode `0600`；owner `cczij`；size `21355`。
- 修改前紧邻复核与开工身份完全一致；未命中 `BLOCKED_CONCURRENT_DRIFT`。

## 3. 实际动作

1. 对用户点名的 4 个路径执行 lstat、realpath 与 config 精确表头计数：四个文件系统路径均为 `ENOENT`；开工 live 中仅 `/Users/USER/Documents/原点向量` 精确块存在，其余 3 个点名块已为 0。
2. 创建 secure backup：`/Users/USER/.codex/secure-backups/20260809T163214+0800/config.toml`。父目录与时间戳目录 `0700`，文件 `0600`，owner `cczij`；SHA-256 与开工 live 相同，`cmp` 一致。备份原文未进入本任务 outputs。
3. 官方 npm registry 只读复核：`https://registry.npmjs.org/shadcn/latest` 返回 `4.16.2`，manifest 版本一致，integrity、40 位 shasum 与官方 HTTPS tarball 字段均存在。未安装、未执行包、未落盘 registry 正文。
4. 仅使用 `apply_patch` 修改 live 正文：
   - 默认模型 `gpt-5.6-sol` 改为 `gpt-5.6-terra`；effort 保持 `medium`。
   - 顶部说明改为“普通执行 Terra/medium；PMO/高风险由创建任务按 AGENTS 第 3 节显式路由”。
   - 删除唯一命中的 `/Users/USER/Documents/原点向量` 两行精确 trusted project 块。
   - shadcn 参数由 `shadcn@latest` 改为 `shadcn@4.16.2`。
5. C-025 trusted code path 两处均保持逐字节不变。

## 4. 第一性原理与对抗式审查

根目标是让普通执行默认路由符合最新全局规则，同时不覆盖并发变化、不扩大 trusted 范围、不泄漏秘密、不删错项目、不中断现有工具运行。

| 前提/攻击或误用 | 阻断证据 | 残余风险 | 修正与闭环 |
| --- | --- | --- | --- |
| 用旧审计快照覆盖新 live | 开工与修改前均核 SHA/mtime/inode/mode/owner；两次完全一致 | 应用可能在任务结束后再次重写 | 结束复核身份；独立验收再次取 live SHA |
| config 原文或 secret 泄漏 | outputs 只含安全字段、节/键状态、路径、哈希；备份只在 secure-backups | 未知键名未来可能带敏感语义 | 输出敏感扫描；独立审查禁止展开 backup/live 原文 |
| 删除名称相似但未授权 project | 只匹配 4 个完整表头；仅 1 个命中被删；其他 project 字节保持 | 原审计另有 3 个失效块仍残留 | 不偷步删除；由 PMO 取得精确新增授权后另修 |
| 动态 `@latest` 引入浮动包 | 官方 registry 当前版本、integrity、shasum、HTTPS tarball 字段复核 | `npx` 首次调用仍可能联网取得固定包 | C-011 的版本漂移关闭；实际调用仍受安装/联网授权门禁约束 |
| 广域 trusted path 给被导入代码能力 | 本机 ChatGPT 打包代码显示：绝对路径表用于判定 imported code 是否注入 trusted module capabilities | 应用同时自动把 Codex home 与 bundled Node modules 写入两个配置位置；删改的功能兼容性与持久性未证 | 保持两处不变，标 C-025 `RESIDUAL_UNCERTAIN`；需产品级权威说明或隔离实验后再修 |
| TOML 语法或未知字节被破坏 | 备份经 3 个精确替换后与 live 逐字节相等；`codex features list` 成功读取 | 自检不是独立验收 | 独立审查复算期望补丁和加载检查 |
| 应用在验证中自动重写 | Codex 配置读取后 SHA/inode/mode/owner 未变化 | 更长时间窗口未证 | PMO 独立验收时再次检查；若漂移按并发漂移处理 |

## 5. 验证结果

- 期望补丁源命中数：`1,1,1`；期望结果与 live `byte-exact = true`。
- Codex 自带配置读取：`codex-cli 0.147.0-alpha.6.5` 的 `codex features list` 退出 0。
- 默认：Terra 计数 1；medium 计数 1；旧默认 Sol 顶层目标已移除。其他兼容模型声明由逐字节期望比较证明未改。
- 路由注释：第 3 节新说明计数 1；旧“第 6 节”说明计数 0。
- 用户点名 4 个 project 表头：分别为 `0/0/0/0`。
- shadcn：`@latest=0`；`@4.16.2=1`。
- trusted code path：2 个条目；与开工备份逐字节一致。
- live 结束身份：SHA-256 `83b7b79ecda5e2ac2b1d04115f8e854c8994b142720d4880a87f4f4c7be6f4f2`；mtime `2026-08-09T16:33:22+0800`；inode `55057589`；mode `0600`；owner `cczij`；size `21306`。
- backup 结束身份：SHA-256 `31a30d48b896e9b5be1cc502bbb945a15c15ede296e63d0d786ba247bdf79aad`；mode `0600`；owner `cczij`；与开工身份相同。
- manifest SHA-256：`15310cd9a673973bd1a5a5e01bdfac13097c0b7c9119cc30a78e20e50256a605`。

## 6. 冲突项状态

| ID | Owner 自检状态 | 结果 |
| --- | --- | --- |
| C-006 | `CLOSED_PENDING_REVIEW` | config 默认新执行任务已为 Terra/medium；不把它当实际运行态证明 |
| C-011 | `CLOSED_PENDING_REVIEW` | 动态版本归零并固定 4.16.2；实际 npx 联网/取得包仍需授权 |
| C-015 | `CLOSED_PENDING_REVIEW` | 按原审计映射，错误 AGENTS 第 6 节注释已改第 3 节 |
| C-025 | `RESIDUAL_UNCERTAIN` | 按原审计映射，广域 trusted code path 两处保持；安全语义已证，兼容/自动写回使收紧方案未证 |
| L-001 | `RESIDUAL_SCOPE_MISMATCH` | 本次点名 4 块均为 0；原审计另列的 `原点向量SEO`、`原点向量官网重塑`、`原点向量官网V2` 三块仍存在且对应目录为 ENOENT，但不在本次允许删除清单 |

说明：本次任务 E 把广域 trusted code path 称为 C-015；原审计冲突矩阵实际把错误章节注释编号为 C-015、广域 trusted code path 编号为 C-025。本报告按原审计编号记账，同时执行了 E 的语义要求。

## 7. 偏差与未触碰范围

- 网页连接器直开 npm registry 被平台 safe-open 拒绝；按 agent-reach 降级为只读 curl，第二次取得最小元数据。未安装任何工具。
- secure backup 首次因父目录不存在而 mkdir 失败，未产生文件；随后仅创建授权的 secure-backups 父目录与唯一时间戳目录。
- 第一次 apply_patch 因下一节上下文判断错误而原子失败；复核 live SHA 未变后，缩小到精确块重试成功。不存在部分补丁。
- 未触碰 AGENTS、memory、SOP、Skill、项目文件、plugin cache、凭据/SSH、远端/生产、其他 MCP/notice/project、secret/token/env 值、Git、sudo、migration、账号/权限。未使用 Reasonix，未派子 agent。
- 实际运行 model/effort/额度无可用运行态读取面，状态为 `未确认`；静态 config 仅代表默认目标。

## 8. 回滚与下一步

回滚材料为 secure backup，但不得由本执行 Owner擅自覆盖 live；若需回滚，先由 PMO 明确授权并重新核 live 身份，避免覆盖后续变更。

独立审查应：复核 live/backup/output SHA；重算脱敏期望补丁；确认 L-001 清单冲突的处置授权；对 C-025 取得产品级语义/兼容证据或做隔离实验；不得把本报告的 Owner 自检改写成最终验收通过。
