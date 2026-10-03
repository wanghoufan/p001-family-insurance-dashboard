# AGENTS.md — 项目总入口

<!-- ORCA-RULES-BLOCK:BEGIN -->
<!-- 本区块由治理母版 scripts/sync-old-projects.sh 于 2026-09-29 注入；只增不删，可重复运行原地更新。 -->
<!-- 本项目 AGENTS.md 的其余内容（项目专属规矩）保持原样，冲突时以本区块为准。 -->
## ORCA 规则增量（母版 2026-09-29-产品验收）

> 本区块只写**对外通用**的机制增量；派工细节见 `docs/roles/`，账本口径见下方条目。

- **产品验收（2026-09-28 定）**：开发完成的判断来自**用户可见要求的覆盖证据**，不只看单测／构建／代码审查／工具调用成功。
  - 验收标准写在计划里：Phase1 给每条用户可见要求编一条可观察可测的**验收条目（AC）**，并标出**关键 AC**（对应 P0／blocking P1／核心用户路径／必要视觉交互呈现）；**关键 AC 集合不得为空**。
  - 证据落 `docs/qa/` 的**产品验收追踪矩阵**（照 `docs/qa/BUGS.template.md` 同名节）。
  - **不可放行三情形**（命中任一不得判 `PASS`）：①关键产品 DoD／AC 未测；②关键任务涉及的**每个**可见操作控件未实际点击并观察到页面／锚点／状态变化（只验 `href` 存在不算）；③验收证据缺失。
  - 视觉验收最小覆盖：关键用户任务逐条走通、按项目要求检查桌面与窄屏、用边界样本（奇偶条目数／长标题长正文／空状态）检验对齐·换行·裁切·溢出·可读性、留真实浏览器截图。
  - **用户签收**：发布类型为**首次发布**的，用户签收通过才算完成（签收前状态记未完成）；迭代更新与局部修复不强制签收。签收属 **Human Gate 范畴（用户参与）**，**不是新增 QA Gate**。
- **体系更新三件套（2026-09-29 定）**：①本项目规则文件改动后与母版对齐（用 `bash scripts/sync-old-projects.sh` 或按《迁移整理提示词》取包，**备份不覆盖**）；②账本内容**不重写**（实绩历史），只做 schema 校验 `node scripts/model/check-ledger.mjs docs/model`（须 `LEDGER-OK`）；③**HANDOFF 记一行**。**老项目无两包概念，故母版的「同步两包＋更新对外概览」不适用。**
- **派工跨目录禁令（2026-09-29 定）**：派 opencode 通道角色（supervisor／neat-freak／experience-recorder）时，任务里读写本仓以外目录（如 `/tmp`、`1.Active/` 等）会被 `external_directory` 权限自动拒、步骤静默失败，可能让角色误报已做也易反复盲试烧额度（禁盲试）；派单前处置二选一——①临时文件改到仓内已 gitignore 的 `temp/`，②先取得用户授权；codebuddy／codex 通道无此限制。
- **红线（2026-09-29 增补）**：产品验收未落盘或关键 AC 未测、不得报完工/收工；首次发布未取得用户签收、不得报完工/收工。
- **本项目迁移状态**：`docs/model/GOVERNANCE-STATE.json`（`rules_version`／`synced_at`／`project_phase_field`／`task_ledger_rows`／`agents_needs_manual_merge`／`product_acceptance_ac_added`）。
- **存量项目待办（不自动做，需项目 TM 判断）**：本项目实绩 Plan 需补「视觉与交互验收标准（AC 编号）＋关键 AC 集合＋发布类型」，否则新规则下收尾会被判**计划缺项**；完成后把 `product_acceptance_ac_added` 置 `true`。
<!-- ORCA-RULES-BLOCK:END -->


本文件是「家庭保单数据看板」项目的**唯一总入口、公共规则与文档路由**。
各角色长规则独立放在 `docs/roles/`；当前任务事实只记录在 `docs/pm/PLAN.md`。

用户最新要求始终优先。

---

## 一、项目档案

- **项目名称**：家庭保单数据看板（family-insurance-dashboard）
- **一句话定位**：纯前端、单文件、离线可用的家庭保单管理看板
- **当前状态**：MVP 已完成并验证（验收项 A1–A6 已通过，见 `docs/pm/PLAN.md`）
- **技术栈**：
  - 原生 HTML / CSS / JavaScript **单文件**，零依赖、零构建、不联网
  - 数据持久化：保单字段存浏览器 `localStorage`；合同附件存 `IndexedDB`
  - 内置脱敏示例数据；内置 Excel 模板（SpreadsheetML `.xls`，Base64 内嵌）
- **目录约定**：
  - `src/`：产品源码（当前为单文件 `src/family-insurance-dashboard.html`）
  - `assets/`：README 插图（3 张脱敏示意图 SVG）
  - `scratch/`：临时 / 辅助脚本、一次性分析、中间文件（**不进 Git**）
  - `docs/`：角色规范、当前任务、QA、审查、交接
  - `prompts/`：7 个角色启动提示词
- **常用命令**：
  - 打开产品：浏览器直接打开 `src/family-insurance-dashboard.html`
  - 本地预览：`cd src && python -m http.server 8000` → http://localhost:8000/family-insurance-dashboard.html
  - 导出 Excel 模板 / JSON、导入恢复：均在看板界面内操作
- **隐私红线**：真实保单数据只在本机浏览器，**绝不联网上传**；`.env*`、密钥、Token 不得提交

---

## 二、角色体系（多 Agent 协作）

常驻 4 角色 + 复杂任务规划 1 + 修复 1 + 收尾 1，共 **7 个**：

1. `【计划】` 技术规划师（Planner）— `docs/roles/planner.md`
2. `【开发】` 开发实现工程师（Builder）— `docs/roles/builder.md`
3. `【审查】` 代码审查工程师（Code Reviewer）— `docs/roles/code-reviewer.md`
4. `【测试】` 质量测试工程师（QA / Test Agent）— `docs/roles/qa.md`
5. `【产品】` 产品体验审查员（Product Reviewer）— `docs/roles/product-reviewer.md`
6. `【修复】` 开发实现工程师（集中读取审查 / QA / 产品报告后修复）
7. `【收尾】` neat-freak

启动提示词见 `prompts/`（00 合集 + 01–07 各角色）。日常启动 Agent 时只发送
「中括号关键词 + 角色 + 当前任务」，Agent 自行读取本文件与对应角色规范。

---

## 三、文档路由（同类信息只维护一个权威位置）

| 信息 | 权威位置 |
|---|---|
| 项目入口 / 公共规则 / 文档路由 | 根目录 `AGENTS.md`（本文件） |
| 当前需求与验收标准 | `docs/pm/PLAN.md` |
| 角色长规则 | `docs/roles/*.md` |
| 长期核心回归基线 | `docs/qa/QA_CHECKLIST.md` |
| Bug 记录 | `docs/qa/BUGS.md` |
| 代码审查发现 | `docs/review/CODE_REVIEW.md` |
| 产品体验建议 | `docs/review/PRODUCT_BACKLOG.md` |
| 历史协作协议 / 长会话交接 | `docs/handoff/HANDOFF.md` |
| 角色启动提示词 | `prompts/` |

---

## 四、公共规则与约束

- Agent 开始工作前**必须先读取本文件（AGENTS.md）**。
- 只读取当前任务真正需要的角色规范和项目文档，避免无关上下文膨胀。
- 禁止随意新增重复的项目管理 Markdown；同类信息只维护上表中的一个权威位置。
- 临时脚本、实验副本、一次性分析、截图和中间文件**统一放 `scratch/`，且不进 Git**。
- `.env*`、密钥、Token、私密配置不得提交。
- 开发者自测**不能替代独立 QA**；QA 不得仅凭 build / lint / 单测 / 主流程成功就判定完整测试通过。
- 产品体验审查**不能退化成传统代码审查**。
- `【计划】` 完成 `PLAN.md` 后必须检查是否需要建立 / 补充 `QA_CHECKLIST.md`。
- Bug 进入 `docs/qa/BUGS.md`；代码审查进入 `docs/review/CODE_REVIEW.md`；产品建议进入 `docs/review/PRODUCT_BACKLOG.md`。
- `【修复】` 优先处理 Code Review / 审查的 **P0、P1** 和产品体验 **P1**；P2 / P3 / Future 默认不自动开发。
- 开发修复后先标记 `VERIFY`，**未经独立 QA 回归不得直接标记 `CLOSED`**。
- neat-freak **不在每次小改动后运行**，只用于较大阶段完成、MVP / Release、长会话交接、文档失配或最终交付。
- 提交、发布、依赖安装或外部系统操作仍需**用户明确授权**（本项目默认不自动提交 / 推送）。

---

## 五、标准流程

```
用户提出任务
→ 【计划】更新 docs/pm/PLAN.md
→ 【开发】实现
→ 【审查】代码审查
→ 【测试】QA
→ 【产品】产品体验审查
→ 【修复】集中修复 P0/P1
→ 【测试】QA 回归
→ 重要阶段：【收尾】neat-freak，并更新 docs/handoff/HANDOFF.md
```

---

## 六、下一步

1. 以本结构（AI编程项目模板-v2.2 多 Agent 协作）作为后续协作基线。
2. 新功能开发前由 `【计划】` 更新 `PLAN.md` 并同步 `QA_CHECKLIST.md`。
3. 重要阶段结束执行 `【收尾】neat-freak`，更新 `docs/handoff/HANDOFF.md`。

---

## 七、V2.3.1 治理迁移桥接（Pilot C）

本项目采用“并存式”治理迁移：本文件仍是产品规则、隐私红线与现有角色体系的唯一权威入口；
`docs/governance/`、`docs/workflow/`、`docs/model/`、`docs/learning/`、`scripts/`、
`tests/governance/` 与 `skills/` 提供 V2.3.1 的任务编排、运行态证据和验收能力。

- 不以模板的根 `AGENTS.md` 覆盖本文件，也不覆盖 `docs/roles/`、`prompts/`、`src/`。
- 所有 Worker 派工仍必须遵守本项目的离线、隐私和文件边界；真实保单、身份证号与合同附件不得被读取、导出、上传或写入治理回执。
- 治理运行态验证只允许使用仓库内脱敏示例数据或专门创建的测试数据；浏览器 `localStorage` 与 `IndexedDB` 是用户私有数据边界。
- 项目的初始治理来源和当前适配状态记录在 `docs/governance/PROJECT_GOVERNANCE_ORIGIN.yaml` 与 `PROJECT_GOVERNANCE_APPLIED.yaml`；适配差异见 `MIGRATION_ADAPTATION.yaml`。
