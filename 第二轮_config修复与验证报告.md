# config 第二轮精确返工与验证报告

建议状态：`READY_FOR_INDEPENDENT_RE_REVIEW`（执行 Owner 自检，非最终验收）  
日期：2026-08-09（Asia/Shanghai）  
唯一 live 修改对象：`/Users/USER/.codex/config.toml`

## 1. 结论

已从第二轮开工 live SHA-256 `f17addeec3f28929ff16bc0a1cb4c55daebb480bf53f524c916007828914d424` 出发，仅删除三个经独立验收确认的旧组织 active project trusted 块：

- `/Users/USER/Documents/原点向量SEO`
- `/Users/USER/Documents/原点向量官网重塑`
- `/Users/USER/Documents/原点向量官网V2`

三个路径 lstat 均为 `ENOENT`，三个精确表头在修改前各命中 1 次且块内仅有 `trust_level` 键；修改后均为 0。没有删除或修改任何其他 project/trusted 块，C-025 的两个 trusted code path 保持逐字节不变。

## 2. 独立验收输入

- 《全局治理修复独立验收报告.md》SHA-256：`3574883f1e391cdf52f76b00ef6d4748d55e06341c4981ab0cfe513efc2a1c7b`，完整读取，结论 `FAIL_REWORK`。
- 《一屏独立验收结论.md》SHA-256：`08f658fad4655aa6cb3efffd69a4d4ca2fd340640405527521f14cf1ec274396`，完整读取。
- `AGENTS.override.md` 不存在。本轮未修改 AGENTS、memory、SOP、Skill、plugin 或其他 live 文件。

## 3. 开工身份与第二备份

- 开工 live：SHA-256 `f17addeec3f28929ff16bc0a1cb4c55daebb480bf53f524c916007828914d424`；mtime `2026-08-09T16:39:45+0800`；inode `55114685`；mode `0600`；owner `cczij`；size `21437`。
- 开工、备份前、补丁前身份三次一致，未命中 `BLOCKED_CONCURRENT_DRIFT`。
- 第二备份：`/Users/USER/.codex/secure-backups/20260809T165626+0800/config.toml`；SHA 与开工 live 相同；时间戳目录 `0700`、文件 `0600`、owner `cczij`；`cmp` 一致。config 原文未进入 outputs。

## 4. `83b7…` 到 `f17a…` 的脱敏结构变化

使用第一轮 secure backup 在内存中重建 `83b7b79e…6f4f2`，重建 SHA 与第一轮报告值完全一致，再与第二轮开工 `f17addee…14d424` 比较：

- 新增节：1 个，即独立验收任务目录的 project 节。
- 新增键：该新增节中的 `trust_level` 1 个。
- 删除节：0。
- 既有共享节的键集合变化：0。
- 既有共享键值变化：0。
- 行数：`652 → 655`。

因此该漂移在结构上仅表现为新增独立验收工作区 project 登记，核心修复语义没有变化。这与应用登记新任务项目块的行为相符，但本轮没有直接因果证据，故原因明确记为 `CAUSE_UNCONFIRMED`，不写成“已确认自动登记”。

## 5. 第一性原理与对抗式检查

根目标：消除能在旧同名目录重现时重新获得 trusted 路由的三个 active 块，同时绝不扩大到其他项目或 C-025。

| 攻击/误用 | 阻断证据 | 残余风险与闭环 |
| --- | --- | --- |
| 用旧快照覆盖当前 app 登记 | 以 `f17a…` 为新基线；修改前连续身份一致 | 独立复验再次核 live SHA；若变化按并发漂移处理 |
| 删除相似名称或其他 project | 三个完整表头各命中 1；唯一期望 diff 只删对应两行块 | 从第二备份重建后与 live 逐字节相等 |
| 借返工处理 C-025 | 第二备份与 live 的两个 trusted code path 行逐字节一致 | C-025 继续由产品级语义/隔离实验处理，本轮不关闭 |
| 破坏 Terra/medium、第三节或 shadcn | 修改后定向计数与 Codex loader 检查 | 独立验收复算同一计数 |
| 输出秘密或 env 值 | 报告仅含路径、键名、计数、状态和 SHA | 结束敏感值复现扫描必须为 0 |
| 应用在加载时再次写回 | Codex loader 前后 live SHA/inode/mode/owner 不变 | 更长时间窗口仍由独立验收重新取证 |

## 6. 验证结果

- 第二备份中三个目标块源命中数：`1,1,1`。
- 第二备份构造唯一期望删除后，与 live `byte-exact = true`。
- 三个目标表头修改后：`0,0,0`。
- 默认：`gpt-5.6-terra` 计数 1；`model_reasoning_effort = medium` 计数 1。
- AGENTS 第 3 节路由注释计数 1。
- shadcn：`@4.16.2=1`；`@latest=0`。
- C-025 trusted code path：2 个；与第二备份逐字节一致。
- Codex 无副作用配置加载：`codex features list` 退出 0。
- 加载后 live 身份：SHA-256 `89b4e0c4fa15709ce5e3045273bb80b41a1624735b3d473ec7bce80633f32a93`；mtime `2026-08-09T16:56:41+0800`；inode `55114685`；mode `0600`；owner `cczij`；size `21192`。
- 第二轮 manifest SHA-256：`53185b40413723af72e33ef437d1bc70eaf15f9561f3353937882f836bc8aa40`。

## 7. 未触碰与下一步

未安装、未联网、未中断应用、未部署、未使用 Reasonix、未派 agent；未改 plugin、memory、AGENTS、SOP、Skill、项目文件、凭据/SSH、远端、生产、Git、migration、账号/权限；未处理 C-025。

本执行 Owner 只建议 `READY_FOR_INDEPENDENT_RE_REVIEW`。下一步由 PMO 新任务只读复核最新 live/backup/output SHA、三块为 0、唯一期望 diff、loader 与权限；不得把本报告自检当作最终验收。
